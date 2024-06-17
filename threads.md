# Threads in Android with Java

In Android, a <b>Thread</b> represents a concurrent unit of execution within a process. Threads have their own call stack but can share their state with other threads in the same process, allowing them to access the same memory area. They are primarily used for performing operations in the background to keep the main UI thread responsive.

## Key Concepts

## 1. Thread

<ul>
  <li>Definition: A Thread is a basic unit of execution that runs independently.</li>
  <li><b>Memory Sharing:</b> Threads within the same process share the same memory space, which allows them to share data but requires careful synchronization to avoid conflicts.</li>
   <li><b>Usage:</b> Commonly used for tasks such as network requests, file I/O, or computations that should not block the UI thread.</li>
</ul>

## 2. UI Thread (Main Thread)

<ul>
  <li><b>Definition:</b> The main thread in Android is responsible for handling UI operations and user interactions.</li>
  <li><b>Thread Safety:</b> UI operations are not thread-safe and must be performed on the UI thread to avoid crashes and ANR (Application Not Responding) errors.</li>
  <li><b>Impact:</b> Blocking the UI thread can lead to a frozen UI and a poor user experience.</li>
</ul>

# Managing Threads

To manage threads effectively in Android using Java, several classes and constructs are available:

## 1. Handler, Looper, and MessageQueue

<ul>
  <li><b>Handler:</b> Handles messages and runnables sent to it by posting them to the associated thread's message queue.</li>
  <li><b>Looper:</b> Runs in a thread and manages its message queue, processing messages sequentially.</li>
  <li><b>MessageQueue:</b> Holds a list of messages that are dispatched to a Looper to process.</li>
</ul>

<pre>
  <code>
    class ExampleThread extends Thread {
    private Handler handler;

    @Override
    public void run() {
        Looper.prepare();
        handler = new Handler(Looper.myLooper());
        Looper.loop();
    }

    public void postTask(Runnable runnable) {
        handler.post(runnable);
    }
}

  </code>
</pre>

## 2.AsyncTask

<ul>
  <li><b>Definition:</b> AsyncTask simplifies background tasks by handling thread management and providing methods to publish progress and results on the UI thread.</li>
  <li>Lifecycle: AsyncTask includes methods like <b>onPreExecute (UI thread)</b>, <b>doInBackground (background thread)</b>, and <b>onPostExecute (UI thread)</b>.</li>
  <li><b>Deprecation:</b> Deprecated in Android API level 30 due to limitations and potential misuse.</li>
</ul>

<pre>
  <code>
    class ExampleAsyncTask extends AsyncTask<Void, Void, String> {

    @Override
    protected void onPreExecute() {
        // Runs on UI thread before doInBackground
    }

    @Override
    protected String doInBackground(Void... voids) {
        // Background thread
        return "Result";
    }

    @Override
    protected void onPostExecute(String result) {
        // Runs on UI thread with result from doInBackground
    }
}

  </code>
</pre>

## 3. Loader

<ul>
  <li><b>Definition:</b> Loaders are used to load data asynchronously in activities or fragments, managing data caching and ensuring data is loaded once and only once.</li>
  <li><b>Lifecycle:</b> Loaders are automatically managed by the system and can handle reloading data when necessary, such as after a configuration change.</li>
</ul>

<pre>
  <code>
    public class ExampleLoader extends AsyncTaskLoader<String> {

    public ExampleLoader(Context context) {
        super(context);
    }

    @Override
    public String loadInBackground() {
        // Background data loading
        return "Data Loaded";
    }

    @Override
    protected void onStartLoading() {
        // Force the loader to start loading
        forceLoad();
    }
}

  </code>
</pre>

## To use this loader in an activity or fragment:

<pre>
  <code>
    public class ExampleActivity extends AppCompatActivity implements LoaderManager.LoaderCallbacks<String> {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_example);

        // Initialize and start the loader
        getSupportLoaderManager().initLoader(0, null, this);
    }

    @Override
    public Loader<String> onCreateLoader(int id, Bundle args) {
        return new ExampleLoader(this);
    }

    @Override
    public void onLoadFinished(Loader<String> loader, String data) {
        // Handle the loaded data
        textView.setText(data);
    }

    @Override
    public void onLoaderReset(Loader<String> loader) {
        // Handle the loader reset
    }
}

  </code>
</pre>

## 4. ThreadPoolExecutor

<ul>
  <li><b>Definition:</b> ThreadPoolExecutor manages a pool of threads for executing tasks concurrently, providing more control over thread management compared to using individual threads.</li>
  <li><b>Configuration:</b> You can configure parameters such as core pool size, maximum pool size, keep-alive time, and the task queue type.</li>
</ul>

<pre>
  <code>
    ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2, 4, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<>()
);

executor.execute(() -> {
    // Background task
    String result = performBackgroundTask();
    runOnUiThread(() -> {
        // Update UI with the result
        textView.setText(result);
    });
});

private String performBackgroundTask() {
    // Simulate delay
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "Background Task Result";
}

  </code>
</pre>

