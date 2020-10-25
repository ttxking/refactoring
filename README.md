## Refactoring Examples

### PA4-flashget

From my Flashget code : https://github.com/ttxking/PA4-flashget

#### FileChooser() method

In the `src/flashget/URTToFileHandler.java` class    
https://github.com/ttxking/PA4-flashget/blob/master/src/flashget/URLToFileHandler.java

* Refactoring: rename method. `FileChooser()` violates the Java naming convention.    
```public File fileChooser(File outputFile, Label fileName , TextField URLField)```

#### initialize() method

In the `src/flashget/Controller.java` class  

consider this code:
``` java
    @FXML
    public void initialize() {

        downloadButton.setOnAction(this::downloadWorker);
        cancelButton.setOnAction(this::stopWorker);
        clearButton.setOnAction(this::clearHandler);



        ProgressBarArray = new ProgressBar[5]; //5 is number of Threads and progressbar
        ProgressBarArray[0] = threadPB1;
        ProgressBarArray[1] = threadPB2;
        ProgressBarArray[2] = threadPB3;
        ProgressBarArray[3] = threadPB4;
        ProgressBarArray[4] = threadPB5;

    }
```

* Refactoring Signs:
    * duplicate Code of ProgressBarArray. Should assign all at once.
    
* Refactoring: append all progressBar at once
    ``` java
      ProgressBarArray = new ProgressBar[]{threadPB1,threadPB2,threadPB3,threadPB4,threadPB5};
    ```
  

#### Download() method
In the `src/flashget/DownloadExecutor.java`

consider this code:
``` java
    public void Download(....) {
         int threadUsed;
 
         // check if file is larger than 50MB (52,428,800 in binary)
         if (length <= 52_428_800) { // less than 50 MB
             threadUsed = 1; // 1 thread
         } else { // more than 50 mb
             threadUsed = 5; // 5 thread
         }
 
         // size of each thread
         long chunkSize = 4096 * 4;
         long chunkNumber = (long) (Math.ceil(length / chunkSize));
         size = (chunkNumber / threadUsed) * chunkSize;
```
* Refactoring Signs:
    * There is a usage of magic number
    * A method contains too many lines of code. The download method create listener, download tasks and update progress bar.
    This seems to be a lot of works.
    * integer division in floating-point context

    
* Refactoring: replace Magic Number with Symbolic Constant and remove math.ceil
``` java
        
        static final long fiftyMB = 52_428_800
        static final int minthread = 1
        static final int maxthread = 2
        static final long chunkSize = 4096 * 4;

        public void Download(....)

        if (length <= fiftyMB) { // less than 50 MB
            threadUsed = minthread; // 1 thread
        } else { // more than 50 mb
            threadUsed = maxthread; // 5 thread
        }

        // size of each thread
        long chunkNumber = (length / chunkSize));
        size = (chunkNumber / threadUsed) * chunkSize;
        
```

* Refactoring: Extract method

    * updateProgress method - update the progress bar
    ``` java
        private void updateProgress(Label threadsLabel, ProgressBar[] progressBarArray, ProgressBar progressBar, int threadUsed) {
            if (threadUsed == 5) {
                // update the main progress bar according to the sub-thread
                progressBar.progressProperty().bind(tasklist.get(0).progressProperty().multiply(0.2).add(
                        tasklist.get(1).progressProperty().multiply(0.2).add(tasklist.get(2).progressProperty().multiply(0.2).add(
                                tasklist.get(3).progressProperty().multiply(0.2).add(tasklist.get(4).progressProperty().multiply(0.2))
                        ))
                ));
            } else {
                progressBar.progressProperty().bind((tasklist.get(0).progressProperty()));
                for (ProgressBar pb : progressBarArray) {
                    pb.setVisible(false);
                }
                threadsLabel.setVisible(false);
            }
        }
    ```
     * downloadTask method - create download task

``` java
    private void downloadTask(URL url, long length, File outputFile, ProgressBar[] progressBarArray, int threadUsed, ChangeListener<Long> changeListener, ChangeListener<String> statusListener) {
            // create a executor service
            ExecutorService executor = Executors.newFixedThreadPool(threadUsed);
    
            // create download task
            for (int i = 0; i < threadUsed; i++) {
    
                if (i == threadUsed - 1) { // last thread
                    task = new DownloadTask(url, outputFile, size * i, length - (size * i)); // last thread handle the rest
                } else {
                    task = new DownloadTask(url, outputFile, (size * i), size);
                }
    
                tasklist.add(task);
    
                // add observer (ChangeListener) of the valueProperty
                tasklist.get(i).valueProperty().addListener(changeListener);
    
                // add it observer (statusListener) of the messageProperty
                tasklist.get(i).messageProperty().addListener(statusListener);
    
                // update the progress bar whenever the worker updates progress
                progressBarArray[i].progressProperty().bind(tasklist.get(i).progressProperty());
    
                // Start executing the task
                executor.execute(tasklist.get(i));
    
                System.out.println("Stating thread" + (i + 1));
    
            }
    
            executor.shutdown();
        }
```
