# Uploaded Files

The file uploads in `$_FILES` are available from the Request object’s `getUploadedFiles()` method. This returns an array keyed by the name of the `input` element.

```php
$files = $request->getUploadedFiles();
```

 Each object in the `$files` array is an instance of `Psr\Http\Message\UploadedFileInterface` and supports the following methods:

- getStream()
- moveTo($targetPath)
- getSize()
- getError()
- getClientFilename()
- getClientMediaType()

See the [cookbook](https://www.slimframework.com/docs/v4/cookbook/uploading-files.html) on how to upload files using a POST form.

## 