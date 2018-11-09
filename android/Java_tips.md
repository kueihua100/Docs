# Java tips:
## How to enable java code's Log.isLoggable?

## How to add call stack at java code?
    Add below java code piece:
    private void printCallStack() 
    {
        StackTraceElement[] elements = Thread.currentThread().getStackTrace();
        for (StackTraceElement element : elements) {
            Log.d("Java_StackTrace", element.toString());
        }
    }
