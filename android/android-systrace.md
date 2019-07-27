# Andorid Performance

## Systrace

### Systrace 可追踪什么

```
$ python $ANDROID_HOME/platform-tools/systrace/systrace.py --list-categories
```

![systrace-list-categories](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/systrace--list-categories.jpg)

```
➜  knight git:(master) python $ANDROID_HOME/platform-tools/systrace/systrace.py --list-categories
         gfx - Graphics
       input - Input
        view - View System
     webview - WebView
          wm - Window Manager
          am - Activity Manager
          sm - Sync Manager
       audio - Audio
       video - Video
      camera - Camera
         hal - Hardware Modules
         app - Application
         res - Resource Loading
      dalvik - Dalvik VM
          rs - RenderScript
      bionic - Bionic C Library
       power - Power Management
          pm - Package Manager
          ss - System Server
    database - Database
     network - Network
       sched - CPU Scheduling
        freq - CPU Frequency
        idle - CPU Idle
        load - CPU Load
  memreclaim - Kernel Memory Reclaim
  binder_driver - Binder Kernel driver
  binder_lock - Binder global lock trace

NOTE: more categories may be available with adb root

```

### 开始追踪

```
python $ANDROID_HOME/platform-tools/systrace/systrace.py -o afirezOnTrace.html \
 sched \
 freq \
 idle \
 am \
 wm \
 gfx \
 view \
 binder_driver \
 hal \
 dalvik \
 camera \
 input \
 res

```

![systrace-usecase](https://raw.githubusercontent.com/afirez/sonice/master/android/assets/systrace-usecase.jpg)

### Systrace Help

```

$ python $ANDROID_HOME/platform-tools/systrace/systrace.py -h
```

```
Usage: systrace.py [options] [category1 [category2 ...]]

Example: systrace.py -b 32768 -t 15 gfx input view sched freq

Options:
  -h, --help            show this help message and exit
  -o FILE               write trace output to FILE
  -j, --json            write a JSON file
  --link-assets         (deprecated)
  --asset-dir=ASSET_DIR
                        (deprecated)
  -e DEVICE_SERIAL_NUMBER, --serial=DEVICE_SERIAL_NUMBER
                        adb device serial number
  --timeout=TIMEOUT     timeout for start and stop tracing (seconds)
  --collection-timeout=COLLECTION_TIMEOUT
                        timeout for data collection (seconds)
  -t N, --time=N        trace for N seconds
  --target=TARGET       choose tracing target (android or  linux)
  -b N, --buf-size=N    use a trace buffer size  of N KB
  -l, --list-categories
                        list the available categories and exit

  Atrace options:
    --atrace-categories=ATRACE_CATEGORIES
                        Select atrace categories with a comma-delimited list,
                        e.g. --atrace-categories=cat1,cat2,cat3
    -k KFUNCS, --ktrace=KFUNCS
                        specify a comma-separated list of kernel functions to
                        trace
    --no-compress       Tell the device not to send the trace data in
                        compressed form.
    -a APP_NAME, --app=APP_NAME
                        enable application-level tracing for comma-separated
                        list of app cmdlines
    --from-file=FROM_FILE
                        read the trace from a file (compressed) rather than
                        running a live trace

  BattOr trace options:
    --battor-categories=BATTOR_CATEGORIES
                        Select battor categories with a comma-delimited list,
                        e.g. --battor-categories=cat1,cat2,cat3
    --serial-map=SERIAL_MAP
                        File containing pregenerated map of phone serial
                        numbers to BattOr serial numbers.
    --battor-path=BATTOR_PATH
                        specify a BattOr path to use
    --battor            Use the BattOr tracing agent.

  Ftrace options:
    --ftrace-categories=FTRACE_CATEGORIES
                        Select ftrace categories with a comma-delimited list,
                        e.g. --ftrace-categories=cat1,cat2,cat3

  WALT trace options:
    --walt              Use the WALT tracing agent. WALT is a device for
                        measuring latency of physical sensors on phones and
                        computers. See https://github.com/google/walt
```

## TraceView

## DDMS

.trace和.html均可使用DDMS生成，AndroidStudio3不再有TraceView，可通过android sdk/toos/monitor打开。

### 启动 Monitor

```

$ $ANDROID_HOME/tools/monitor
```