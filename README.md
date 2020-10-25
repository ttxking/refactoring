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
https://github.com/ttxking/PA4-flashget/blob/master/src/flashget/Controller.java

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
      ...
      ProgressBarArray = new ProgressBar[]{threadPB1,threadPB2,threadPB3,threadPB4,threadPB5};
    ```
  

#### Download() method
In the `src/flashget/DownloadExecutor.java`    
https://github.com/ttxking/PA4-flashget/blob/master/src/flashget/DownloadExecutor.java

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
    
        ...
```
* Refactoring Signs:
    * There is a usage of magic number
    * A method contains too many lines of code. The download method create listener, download tasks and update progress bar.
    This seems to be a lot of works.
    * Integer division in floating-point context
    * Contains an anonymous new ChangeListener<>()

    
* Refactoring: replace Magic Number with Symbolic Constant 
    ``` java
            ...
  
            static final long fiftyMB = 52_428_800
            static final int minthread = 1
            static final int maxthread = 2
            static final long chunkSize = 4096 * 4;
    
            public void Download(....)
            ...
    
            if (length <= fiftyMB) { // less than 50 MB
                threadUsed = minthread; // 1 thread
            } else { // more than 50 mb
                threadUsed = maxthread; // 5 thread
            }         
            ...
    ```
  
* Refactoring : Remove math.ceil from integer division
    ``` java
        ...
        // size of each thread
              long chunkNumber = (length / chunkSize));
              size = (chunkNumber / threadUsed) * chunkSize;
        ...
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
 * Refactoring: replace with lambda
     ``` java
            ...
            // update Label that shows byte downloaded
            ChangeListener<Long> changeListener =
                    (observable, oldValue, newValue) -> {
                        if (oldValue == null) {
                            oldValue = 0L; // in case the old value is null set in to zero
                        }
                        byteDownloaded += newValue - oldValue; // byte download on each thread
                        downloadLabel.setText(String.format("%d/%d", byteDownloaded, length)); // update the label
                    };
    
            // update Label that shows status value of downloader
            ChangeListener<String> statusListener =
                    (observable, oldValue, newValue) -> downloadLabel.setText(newValue);
     ```
     