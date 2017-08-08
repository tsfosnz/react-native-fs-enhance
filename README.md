# react-native-fs-enhance
Adding some code to enhance fs

react-native-fs is a good library, but for use, I need something more:

- mkdirs API to add more than one folders same time
- existsSync API to check File existence in sync mode

Here is the code:

### Android

- RNFSManager.java

```java

    @ReactMethod
    public void mkdirs(ReadableArray filepaths, ReadableMap options, Promise promise) {
      String filepath = "";

      try {

        for (int i = 0; i < filepaths.size(); ++i) {
          filepath = filepaths.getString(i);
          File file = new File(filepath);
          // Log.i("RNFS", filepath);

          file.mkdirs();
          boolean exists = file.exists();
          if (!exists) throw new Exception("Directory could not be created");


          file.setReadable(true);
          file.setWritable(true);
          file.setExecutable(true);
        }

        promise.resolve(null);
      } catch (Exception ex) {
        ex.printStackTrace();
        reject(promise, filepath, ex);
      }
    }
   ```
   
   And:
   
   ```java
  @ReactMethod(isBlockingSynchronousMethod = true)
  public Boolean existsSync(String filepath) {
    try {
      File file = new File(filepath);
      return file.exists();
    } catch (Exception ex) {
      ex.printStackTrace();
    }
    return false;
  }
  ```
  
  ### iOS
  
  - RNFSManager.m
  
  ```objc
  RCT_EXPORT_METHOD(mkdirs:(NSArray *)filepaths
                  options:(NSDictionary *)options
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject)
{
    NSFileManager *manager = [NSFileManager defaultManager];
    
    
    for (int i = 0; i < [filepaths count]; i += 1) {
        NSString *filepath = [filepaths objectAtIndex: i];
        NSError *error = nil;
        BOOL success = [manager createDirectoryAtPath:filepath withIntermediateDirectories:YES attributes:nil error:&error];
        
        if (!success) {
            return [self reject:reject withError:error];
        }
        NSURL *url = [NSURL fileURLWithPath:filepath];
        
        if ([[options allKeys] containsObject:@"NSURLIsExcludedFromBackupKey"]) {
            NSNumber *value = options[@"NSURLIsExcludedFromBackupKey"];
            success = [url setResourceValue: value forKey: NSURLIsExcludedFromBackupKey error: &error];
            
            if (!success) {
                return [self reject:reject withError:error];
            }
        }
    }
    
    resolve(nil);
}
```

And

```objc
RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(existsSync:(NSString *)filepath)
{
    BOOL fileExists = [[NSFileManager defaultManager] fileExistsAtPath:filepath];

    return [NSNumber numberWithBool:fileExists];
}
```
