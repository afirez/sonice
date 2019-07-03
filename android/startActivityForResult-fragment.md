# startActivityForResult

![](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/startActivityForResult-fragment.jpg)

~~~
// android.app.Fragment.class#startActivityForResult
// api 28

// Fragment.class
public void startActivityForResult(Intent intent, int requestCode, Bundle options) {
  if (mHost == null) {
    throw new IllegalStateException("Fragment " + this + " not attached to Activity");
  }
  //mHost == Activity.HostCallbacks.class (FragmentHostCallback.class)
  mHost.onStartActivityFromFragment(this, intent, requestCode, options);
}

// Activity.HostCallbacks.class
@Override
public void onStartActivityFromFragment(
  Fragment fragment, Intent intent, int requestCode, Bundle options) {
  Activity.this.startActivityFromFragment(fragment, intent, requestCode, options);
}

// Activity.class
public void startActivityFromFragment(
  @NonNull Fragment fragment,
  @RequiresPermission Intent intent, 
  int requestCode, 
  @Nullable Bundle options) {
  // **fragment.mWho**
  startActivityForResult(fragment.mWho, intent, requestCode, options);
}

// Activity.class
@Override
public void startActivityForResult(
  String who, Intent intent, int requestCode, @Nullable Bundle options) {
  Uri referrer = onProvideReferrer();
  if (referrer != null) {
    intent.putExtra(Intent.EXTRA_REFERRER, referrer);
  }
  options = transferSpringboardActivityOptions(options);
  
  // ar = Instrumentation.execStartActivity
  // 1. ar != null 
  //   --> ActivityThread.sendActivityResult()
  // 2. ar == null 
  //   --> ActivityManager.getService().startActivity() // (Binder IPC)
  //   <-- ActivityThread.ApplicationThread.scheduleSendResult()
  
  Instrumentation.ActivityResult ar =
    mInstrumentation.execStartActivity(
    this, 
    mMainThread.getApplicationThread(), 
    mToken, 
    who,
    intent, 
    requestCode, 
    options);
  
  if (ar != null) {
    // mMainThread == ActivityThread.class
    mMainThread.sendActivityResult(
      mToken, who, requestCode, ar.getResultCode(), ar.getResultData());
  }
  cancelInputsAndStartExitTransition(options);
}

// ActivityThread.class
public final void sendActivityResult(
  IBinder token, 
  String id, 
  int requestCode,
  int resultCode, 
  Intent data) {
  ArrayList<ResultInfo> list = new ArrayList<ResultInfo>();
  list.add(new ResultInfo(id, requestCode, resultCode, data));
   // mAppThread == ActivityThread.ApplicationThread.class
  mAppThread.scheduleSendResult(token, list);
}

// ActivityThread.ApplicationThread.class
public final void scheduleSendResult(IBinder token, List<ResultInfo> results) {
  ResultData res = new ResultData();
  res.token = token;
  res.results = results;
  // ** ActivityThread.H.SEND_RESULT (Handler) **
  sendMessage(H.SEND_RESULT, res);
}

// ActivityThread.H.Class
public void handleMessage(Message msg) { 
  switch (msg.what) {
    case SEND_RESULT:
      // ActivityThread#handleSendResult
      handleSendResult((ResultData)msg.obj);
      break;
  }
}

// ActivityThread.Class
private void handleSendResult(ResultData res) {
  ActivityClientRecord r = mActivities.get(res.token);
  if (r != null) {
    final boolean resumed = !r.paused;
    if (!r.activity.mFinished 
        && r.activity.mDecor != null
        && r.hideForNow 
        && resumed) {
      // We had hidden the activity because it started another
      // one...  we have gotten a result back and we are not
      // paused, so make sure our window is visible.
      updateVisibility(r, true);
    }
    if (resumed) {
      try {
        // Now we are idle.
        r.activity.mCalled = false;
        r.activity.mTemporaryPause = true;
        mInstrumentation.callActivityOnPause(r.activity);
        if (!r.activity.mCalled) {
          throw new SuperNotCalledException(
            "Activity " + r.intent.getComponent().toShortString()
            + " did not call through to super.onPause()");
        }
      } catch (SuperNotCalledException e) {
        throw e;
      } catch (Exception e) {
        if (!mInstrumentation.onException(r.activity, e)) {
          throw new RuntimeException(
            "Unable to pause activity "
            + r.intent.getComponent().toShortString()
            + ": " + e.toString(), e);
        }
      }
    }
    checkAndBlockForNetworkAccess();
    
    // ActivityThread#deliverResults
    deliverResults(r, res.results);
    
    if (resumed) {
      r.activity.performResume();
      r.activity.mTemporaryPause = false;
    }
  }
}

// ActivityThread.Class
private void deliverResults(ActivityClientRecord r, List<ResultInfo> results) {
  final int N = results.size();
  for (int i=0; i<N; i++) {
    ResultInfo ri = results.get(i);
    try {
      if (ri.mData != null) {
        ri.mData.setExtrasClassLoader(r.activity.getClassLoader());
        ri.mData.prepareToEnterProcess();
      }
      
      if (DEBUG_RESULTS) Slog.v(
        TAG,"Delivering result to activity " + r + " : " + ri);
      
      // Activity.dispatchActivityResult
      r.activity.dispatchActivityResult(
        ri.mResultWho,
        ri.mRequestCode, 
        ri.mResultCode, 
        ri.mData);
      
    } catch (Exception e) {
      if (!mInstrumentation.onException(r.activity, e)) {
        throw new RuntimeException(
          "Failure delivering result " + ri + " to activity "
          + r.intent.getComponent().toShortString()
          + ": " + e.toString(), e);
      }
    }
  }
}

// Activity.class
void dispatchActivityResult(
  String who, 
  int requestCode,
  int resultCode, Intent data) {
  
  if (false) Log.v(
    TAG, "Dispatching result: who=" + who + ", reqCode=" + requestCode
    + ", resCode=" + resultCode + ", data=" + data);
  
  mFragments.noteStateNotSaved();
  if (who == null) {
    // ??? 1、 no who: do it by self
    onActivityResult(requestCode, resultCode, data);
  } else if (who.startsWith(REQUEST_PERMISSIONS_WHO_PREFIX)) {
    // ??? 2、 REQUEST_PERMISSIONS_WHO_PREFIX: REQUEST_PERMISSIONS Adapter and do it by self
    who = who.substring(REQUEST_PERMISSIONS_WHO_PREFIX.length());
    if (TextUtils.isEmpty(who)) {
      // ??? 2.1、REQUEST_PERMISSIONS no who: do it by self
      dispatchRequestPermissionsResult(requestCode, data);
    } else {
      Fragment frag = mFragments.findFragmentByWho(who);
      if (frag != null) {
        // ??? 2.1、REQUEST_PERMISSIONS has who: do it by Fragment
        dispatchRequestPermissionsResultToFragment(requestCode, data, frag);
      }
    }
  } else if (who.startsWith("@android:view:")) {
    // ??? 3、who is View: do it by View
    ArrayList<ViewRootImpl> views = WindowManagerGlobal.getInstance().getRootViews(
      getActivityToken());
    for (ViewRootImpl viewRoot : views) {
      if (viewRoot.getView() != null
          && viewRoot.getView().dispatchActivityResult(
            who, requestCode, resultCode, data)) {
        return;
      }
    }
  } else if (who.startsWith(AUTO_FILL_AUTH_WHO_PREFIX)) {
    // ??? 4、who is AutofillManager: do it by AutofillManager
    Intent resultData = (resultCode == Activity.RESULT_OK) ? data : null;
    getAutofillManager().onAuthenticationResult(requestCode, resultData);
  } else {
    Fragment frag = mFragments.findFragmentByWho(who);
    if (frag != null) {
      // frag = Fragment.class
      frag.onActivityResult(requestCode, resultCode, data);
    }
  }
}

// Fragment.class
public void onActivityResult(int requestCode, int resultCode, Intent data) {
  // So nice! Welcome here!
}

~~~