# Playwright File Upload and Download

## 1. Introduction

Playwright provides built-in support for handling file uploads and downloads.

Common file-handling scenarios include:

* Uploading a single file
* Uploading multiple files
* Uploading files through a file chooser
* Downloading files
* Verifying downloaded files
* Renaming downloaded files
* Saving downloads to a specific location
* Validating downloaded file names
* Validating file extensions
* Uploading files in API or UI workflows
* Handling dynamic upload controls

Playwright supports file operations without requiring third-party tools.

---

# 2. File Upload

The simplest way to upload a file is using `setInputFiles()`.

### HTML Example

```html
<input type="file" id="fileUpload">
```

### Playwright Java Example

```java
page.locator("#fileUpload")
    .setInputFiles(Paths.get("src/test/resources/test.pdf"));
```

### Complete Example

```java
import com.microsoft.playwright.*;
import java.nio.file.Paths;

public class FileUploadExample {

    public static void main(String[] args) {

        try (Playwright playwright = Playwright.create()) {

            Browser browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false)
            );

            Page page = browser.newPage();

            page.navigate("https://example.com/upload");

            page.locator("input[type='file']")
                .setInputFiles(
                    Paths.get("src/test/resources/test.pdf")
                );

            browser.close();
        }
    }
}
```

---

# 3. Upload Using a Locator

You can directly use a locator.

```java
page.locator("input[type='file']")
    .setInputFiles(Paths.get("src/test/resources/test.pdf"));
```

Using a role or label is also possible when the application exposes appropriate accessibility information.

```java
page.getByLabel("Upload File")
    .setInputFiles(Paths.get("src/test/resources/test.pdf"));
```

---

# 4. Upload Using `setInputFiles()`

`setInputFiles()` is the preferred approach when a file input element is available.

```java
page.locator("#upload")
    .setInputFiles(Paths.get("src/test/resources/sample.txt"));
```

You can use:

```java
setInputFiles(Path)
```

or:

```java
setInputFiles(Path...)
```

---

# 5. Upload Multiple Files

Playwright supports multiple-file uploads.

### HTML

```html
<input type="file" id="files" multiple>
```

### Java

```java
page.locator("#files").setInputFiles(
    Paths.get("src/test/resources/file1.txt"),
    Paths.get("src/test/resources/file2.txt"),
    Paths.get("src/test/resources/file3.txt")
);
```

Another approach is:

```java
Path file1 = Paths.get("src/test/resources/file1.txt");
Path file2 = Paths.get("src/test/resources/file2.txt");

page.locator("#files")
    .setInputFiles(file1, file2);
```

---

# 6. Upload a Single File

```java
Path file = Paths.get("src/test/resources/resume.pdf");

page.locator("input[type='file']")
    .setInputFiles(file);
```

---

# 7. Upload Different File Types

Playwright does not restrict the file extension during automation.

For example:

```java
page.locator("#upload")
    .setInputFiles(Paths.get("src/test/resources/image.png"));
```

```java
page.locator("#upload")
    .setInputFiles(Paths.get("src/test/resources/document.pdf"));
```

```java
page.locator("#upload")
    .setInputFiles(Paths.get("src/test/resources/data.csv"));
```

```java
page.locator("#upload")
    .setInputFiles(Paths.get("src/test/resources/test.xlsx"));
```

The application itself determines whether the file type is accepted.

---

# 8. Upload Using File Chooser

Some applications do not expose a normal file input directly.

For example, clicking an "Upload" button may open the operating-system file chooser.

Playwright provides `FileChooser` for this scenario.

```java
FileChooser fileChooser = page.waitForFileChooser(
    () -> page.getByRole(
        AriaRole.BUTTON,
        new Page.GetByRoleOptions().setName("Upload")
    ).click()
);

fileChooser.setFiles(
    Paths.get("src/test/resources/test.pdf")
);
```

---

# 9. File Chooser Example

```java
import com.microsoft.playwright.*;
import java.nio.file.Paths;

public class FileChooserExample {

    public static void main(String[] args) {

        try (Playwright playwright = Playwright.create()) {

            Browser browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false)
            );

            Page page = browser.newPage();

            page.navigate("https://example.com");

            FileChooser fileChooser = page.waitForFileChooser(
                () -> page.getByText("Choose File").click()
            );

            fileChooser.setFiles(
                Paths.get("src/test/resources/test.pdf")
            );

            browser.close();
        }
    }
}
```

---

# 10. Why Use `waitForFileChooser()`?

The file chooser event happens as a result of an action.

Therefore, you should wait for the event and perform the triggering action together.

Correct:

```java
FileChooser fileChooser = page.waitForFileChooser(
    () -> page.getByText("Upload").click()
);

fileChooser.setFiles(
    Paths.get("src/test/resources/test.pdf")
);
```

Avoid trying to interact with the operating system's file chooser manually.

---

# 11. Upload and Verify

After uploading, verify that the application processed the file.

```java
Path file = Paths.get("src/test/resources/test.pdf");

page.locator("input[type='file']")
    .setInputFiles(file);

page.getByRole(
    AriaRole.BUTTON,
    new Page.GetByRoleOptions().setName("Upload")
).click();

assertThat(
    page.getByText("File uploaded successfully")
).isVisible();
```

---

# 12. Verify Uploaded File Name

```java
page.locator("input[type='file']")
    .setInputFiles(Paths.get("src/test/resources/test.pdf"));

assertThat(
    page.locator(".file-name")
).containsText("test.pdf");
```

---

# 13. Clear a Selected File

You can remove the selected file using an empty file list.

```java
page.locator("input[type='file']")
    .setInputFiles(new Path[0]);
```

This is useful when testing:

* Remove file
* Replace file
* Clear upload
* Upload validation

---

# 14. Replace an Uploaded File

```java
Path firstFile = Paths.get("src/test/resources/file1.pdf");
Path secondFile = Paths.get("src/test/resources/file2.pdf");

Locator upload = page.locator("input[type='file']");

upload.setInputFiles(firstFile);

upload.setInputFiles(secondFile);
```

The second operation replaces the selected file.

---

# 15. File Upload Validation

A good upload test should validate more than just the presence of the file.

Typical validations include:

```text
File name
File extension
File size
File type
File content
Upload status
Success message
Error message
Maximum file size
Duplicate file handling
Required upload validation
```

Example:

```java
page.locator("#upload")
    .setInputFiles(Paths.get("src/test/resources/test.pdf"));

assertThat(
    page.locator(".upload-status")
).hasText("Upload successful");
```

---

# 16. Testing Invalid File Types

Suppose the application accepts only PDF files.

Upload an invalid file:

```java
page.locator("#upload")
    .setInputFiles(Paths.get("src/test/resources/test.exe"));
```

Verify the validation message:

```java
assertThat(
    page.getByText("Invalid file type")
).isVisible();
```

---

# 17. Testing Maximum File Size

You can use a large test file:

```java
page.locator("#upload")
    .setInputFiles(
        Paths.get("src/test/resources/large-file.pdf")
    );
```

Then verify:

```java
assertThat(
    page.getByText("File size exceeds the allowed limit")
).isVisible();
```

---

# 18. File Download

Playwright provides the `Download` object for handling downloads.

A typical download flow is:

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);
```

Then save the file:

```java
download.saveAs(
    Paths.get("target/downloads/file.pdf")
);
```

---

# 19. Complete Download Example

```java
import com.microsoft.playwright.*;
import java.nio.file.Paths;

public class FileDownloadExample {

    public static void main(String[] args) {

        try (Playwright playwright = Playwright.create()) {

            Browser browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(false)
            );

            Page page = browser.newPage();

            page.navigate("https://example.com/download");

            Download download = page.waitForDownload(
                () -> page.getByText("Download").click()
            );

            download.saveAs(
                Paths.get("target/downloads/test.pdf")
            );

            browser.close();
        }
    }
}
```

---

# 20. Why Use `waitForDownload()`?

A download is triggered asynchronously.

The correct pattern is:

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);
```

This tells Playwright:

1. Wait for the download event.
2. Perform the click.
3. Capture the resulting download.

---

# 21. Get Download File Name

You can retrieve the suggested file name.

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);

String fileName = download.suggestedFilename();

System.out.println(fileName);
```

Example output:

```text
customer-report.pdf
```

---

# 22. Save Download to a Specific Location

```java
Path downloadPath =
    Paths.get("target/downloads/customer-report.pdf");

download.saveAs(downloadPath);
```

---

# 23. Get Download Path

Playwright can provide the temporary path of the downloaded file.

```java
Path path = download.path();

System.out.println(path);
```

However, for test frameworks it is generally better to use:

```java
download.saveAs(...)
```

to place the file in a predictable test directory.

---

# 24. Verify Downloaded File Exists

Using Java NIO:

```java
Path downloadPath =
    Paths.get("target/downloads/test.pdf");

download.saveAs(downloadPath);

assertTrue(Files.exists(downloadPath));
```

Required imports:

```java
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

import static org.junit.jupiter.api.Assertions.assertTrue;
```

---

# 25. Verify Downloaded File Name

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);

assertEquals(
    "customer-report.pdf",
    download.suggestedFilename()
);
```

---

# 26. Verify Downloaded File Extension

```java
String fileName = download.suggestedFilename();

assertTrue(fileName.endsWith(".pdf"));
```

---

# 27. Verify Downloaded File Size

```java
Path downloadPath =
    Paths.get("target/downloads/test.pdf");

download.saveAs(downloadPath);

long fileSize = Files.size(downloadPath);

assertTrue(fileSize > 0);
```

---

# 28. Verify Downloaded File Content

For text files:

```java
Path downloadPath =
    Paths.get("target/downloads/test.txt");

download.saveAs(downloadPath);

String content =
    Files.readString(downloadPath);

assertTrue(content.contains("Test Data"));
```

This is useful for validating:

* CSV
* TXT
* JSON
* XML
* Log files

---

# 29. Download CSV and Validate Content

```java
Download download = page.waitForDownload(
    () -> page.getByText("Export CSV").click()
);

Path path =
    Paths.get("target/downloads/report.csv");

download.saveAs(path);

String content = Files.readString(path);

assertTrue(content.contains("Customer"));
assertTrue(content.contains("Order"));
```

---

# 30. Download JSON and Validate Content

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download JSON").click()
);

Path path =
    Paths.get("target/downloads/data.json");

download.saveAs(path);

String content = Files.readString(path);

assertTrue(content.contains("customerId"));
```

---

# 31. Multiple Downloads

If a page generates multiple downloads, handle each download separately.

```java
Download download1 = page.waitForDownload(
    () -> page.getByText("Download File 1").click()
);

download1.saveAs(
    Paths.get("target/downloads/file1.pdf")
);

Download download2 = page.waitForDownload(
    () -> page.getByText("Download File 2").click()
);

download2.saveAs(
    Paths.get("target/downloads/file2.pdf")
);
```

---

# 32. Download from a Link

A normal download link can be handled the same way.

```java
Download download = page.waitForDownload(
    () -> page.locator("a.download-link").click()
);

download.saveAs(
    Paths.get("target/downloads/file.zip")
);
```

---

# 33. Download with a Button

```java
Download download = page.waitForDownload(
    () -> page.getByRole(
        AriaRole.BUTTON,
        new Page.GetByRoleOptions().setName("Download Report")
    ).click()
);

download.saveAs(
    Paths.get("target/downloads/report.pdf")
);
```

---

# 34. Download with TestNG

```java
@Test
public void verifyFileDownload() throws Exception {

    Download download = page.waitForDownload(
        () -> page.getByText("Download").click()
    );

    Path path =
        Paths.get("target/downloads/report.pdf");

    download.saveAs(path);

    Assert.assertTrue(Files.exists(path));
    Assert.assertTrue(Files.size(path) > 0);
}
```

---

# 35. Upload with TestNG

```java
@Test
public void verifyFileUpload() {

    Path file =
        Paths.get("src/test/resources/test.pdf");

    page.locator("input[type='file']")
        .setInputFiles(file);

    page.getByRole(
        AriaRole.BUTTON,
        new Page.GetByRoleOptions().setName("Upload")
    ).click();

    Assert.assertTrue(
        page.getByText("File uploaded successfully")
            .isVisible()
    );
}
```

---

# 36. Upload and Download in the Same Test

```java
@Test
public void uploadAndDownload() throws Exception {

    Path uploadFile =
        Paths.get("src/test/resources/input.pdf");

    page.locator("input[type='file']")
        .setInputFiles(uploadFile);

    page.getByRole(
        AriaRole.BUTTON,
        new Page.GetByRoleOptions().setName("Upload")
    ).click();

    assertThat(
        page.getByText("Upload successful")
    ).isVisible();

    Download download = page.waitForDownload(
        () -> page.getByText("Download").click()
    );

    Path downloadPath =
        Paths.get("target/downloads/output.pdf");

    download.saveAs(downloadPath);

    Assert.assertTrue(
        Files.exists(downloadPath)
    );
}
```

---

# 37. Handling Downloads in Headless Mode

Downloads work in headless mode.

```java
Browser browser = playwright.chromium().launch(
    new BrowserType.LaunchOptions()
        .setHeadless(true)
);
```

Then:

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);
```

There is no need to manually open the browser's download window.

---

# 38. Download Directory

Create a dedicated directory for test downloads.

Example:

```text
project
│
├── src
│   └── test
│       └── resources
│           ├── test.pdf
│           ├── test.csv
│           └── test.txt
│
├── target
│   └── downloads
│
├── pom.xml
└── playwright.config
```

You can save downloads here:

```java
Paths.get("target/downloads/report.pdf")
```

---

# 39. Create Download Directory Automatically

```java
Path downloadDirectory =
    Paths.get("target/downloads");

Files.createDirectories(downloadDirectory);
```

Then:

```java
download.saveAs(
    downloadDirectory.resolve(
        download.suggestedFilename()
    )
);
```

---

# 40. Reuse the Suggested File Name

Instead of hardcoding the name:

```java
String fileName = download.suggestedFilename();

Path downloadPath =
    Paths.get("target/downloads")
         .resolve(fileName);

download.saveAs(downloadPath);
```

This is useful when the server generates dynamic file names.

---

# 41. Dynamic Download File Name

Suppose the application downloads:

```text
report_2026_08_19.pdf
```

Instead of assuming the exact name:

```java
String fileName = download.suggestedFilename();

Assert.assertTrue(
    fileName.startsWith("report_")
);

Assert.assertTrue(
    fileName.endsWith(".pdf")
);
```

---

# 42. Download Failure Handling

You can inspect the failure state.

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);

String failure = download.failure();

Assert.assertNull(failure);
```

If the download fails, `failure()` can provide information about the failure.

---

# 43. Upload Files from Absolute Path

```java
Path file =
    Paths.get("C:/Automation/TestData/test.pdf");

page.locator("input[type='file']")
    .setInputFiles(file);
```

For Windows paths, Java accepts:

```java
Paths.get("C:/Automation/TestData/test.pdf");
```

Using `/` avoids many escaping issues.

---

# 44. Upload Files Using Relative Paths

Recommended for portable automation projects:

```java
Path file =
    Paths.get("src/test/resources/test.pdf");

page.locator("input[type='file']")
    .setInputFiles(file);
```

This makes the test easier to execute on:

* Developer machines
* CI/CD servers
* Docker containers
* Remote execution environments

---

# 45. Upload Multiple Files Using an Array

```java
Path[] files = {
    Paths.get("src/test/resources/file1.pdf"),
    Paths.get("src/test/resources/file2.pdf"),
    Paths.get("src/test/resources/file3.pdf")
};

page.locator("input[type='file']")
    .setInputFiles(files);
```

---

# 46. Upload File from Test Resources

A clean framework approach is to store test files under:

```text
src/test/resources/testdata/
```

Example:

```text
src/test/resources/testdata/
├── resume.pdf
├── sample.csv
├── customer.json
└── image.png
```

Then:

```java
Path file =
    Paths.get(
        "src/test/resources/testdata/resume.pdf"
    );

page.locator("input[type='file']")
    .setInputFiles(file);
```

---

# 47. Upload Using Classpath Resources

For Maven projects, test resources can be loaded from the classpath.

```java
URL resource =
    getClass()
        .getClassLoader()
        .getResource("testdata/resume.pdf");

Path file =
    Paths.get(resource.toURI());

page.locator("input[type='file']")
    .setInputFiles(file);
```

This is useful when the framework is executed from different environments.

---

# 48. Verify File Input

You can inspect the file input after selection.

```java
String value =
    page.locator("input[type='file']")
        .inputValue();

System.out.println(value);
```

However, the exact value may vary by browser and application implementation, so application-level validation is generally preferred.

---

# 49. Drag-and-Drop File Upload

Some applications provide drag-and-drop upload areas.

The implementation depends on how the application exposes its file input.

If a hidden file input exists, you can often upload directly:

```java
page.locator("input[type='file']")
    .setInputFiles(
        Paths.get("src/test/resources/test.pdf")
    );
```

This is usually more reliable than trying to simulate physical OS-level dragging.

---

# 50. Testing Required File Validation

If a file is mandatory:

```java
page.getByRole(
    AriaRole.BUTTON,
    new Page.GetByRoleOptions().setName("Submit")
).click();

assertThat(
    page.getByText("File is required")
).isVisible();
```

---

# 51. Testing Duplicate File Upload

```java
Path file =
    Paths.get("src/test/resources/test.pdf");

Locator upload =
    page.locator("input[type='file']");

upload.setInputFiles(file);

page.getByRole(
    AriaRole.BUTTON,
    new Page.GetByRoleOptions().setName("Upload")
).click();

upload.setInputFiles(file);

page.getByRole(
    AriaRole.BUTTON,
    new Page.GetByRoleOptions().setName("Upload")
).click();

assertThat(
    page.getByText("File already exists")
).isVisible();
```

---

# 52. File Upload with Page Object Model

Create a page object:

```java
public class UploadPage {

    private final Page page;

    private final Locator fileInput;
    private final Locator uploadButton;
    private final Locator successMessage;

    public UploadPage(Page page) {
        this.page = page;

        fileInput =
            page.locator("input[type='file']");

        uploadButton =
            page.getByRole(
                AriaRole.BUTTON,
                new Page.GetByRoleOptions()
                    .setName("Upload")
            );

        successMessage =
            page.getByText(
                "File uploaded successfully"
            );
    }

    public void uploadFile(Path file) {
        fileInput.setInputFiles(file);
    }

    public void clickUpload() {
        uploadButton.click();
    }

    public boolean isUploadSuccessful() {
        return successMessage.isVisible();
    }
}
```

Test:

```java
@Test
public void uploadFileTest() {

    UploadPage uploadPage =
        new UploadPage(page);

    Path file =
        Paths.get("src/test/resources/test.pdf");

    uploadPage.uploadFile(file);
    uploadPage.clickUpload();

    Assert.assertTrue(
        uploadPage.isUploadSuccessful()
    );
}
```

---

# 53. Download Page Object

```java
public class DownloadPage {

    private final Page page;

    private final Locator downloadButton;

    public DownloadPage(Page page) {

        this.page = page;

        downloadButton =
            page.getByRole(
                AriaRole.BUTTON,
                new Page.GetByRoleOptions()
                    .setName("Download")
            );
    }

    public Download downloadFile() {

        return page.waitForDownload(
            () -> downloadButton.click()
        );
    }
}
```

Test:

```java
@Test
public void downloadFileTest() throws Exception {

    DownloadPage downloadPage =
        new DownloadPage(page);

    Download download =
        downloadPage.downloadFile();

    Path path =
        Paths.get("target/downloads/report.pdf");

    download.saveAs(path);

    Assert.assertTrue(
        Files.exists(path)
    );
}
```

---

# 54. Utility Class for File Upload

```java
public class FileUtils {

    public static void uploadFile(
            Locator locator,
            String filePath) {

        locator.setInputFiles(
            Paths.get(filePath)
        );
    }
}
```

Usage:

```java
FileUtils.uploadFile(
    page.locator("input[type='file']"),
    "src/test/resources/test.pdf"
);
```

---

# 55. Utility Method for Downloads

```java
public static Path saveDownload(
        Page page,
        Locator downloadButton,
        Path directory) throws Exception {

    Files.createDirectories(directory);

    Download download =
        page.waitForDownload(
            downloadButton::click
        );

    Path file =
        directory.resolve(
            download.suggestedFilename()
        );

    download.saveAs(file);

    return file;
}
```

Usage:

```java
Path downloadedFile =
    FileUtils.saveDownload(
        page,
        page.getByText("Download"),
        Paths.get("target/downloads")
    );

Assert.assertTrue(
    Files.exists(downloadedFile)
);
```

---

# 56. Cleanup Downloaded Files

Old files can cause false-positive results.

Before a test:

```java
Path downloadDirectory =
    Paths.get("target/downloads");

Files.createDirectories(downloadDirectory);
```

Delete previous files:

```java
try (Stream<Path> files =
         Files.list(downloadDirectory)) {

    files.forEach(path -> {
        try {
            Files.deleteIfExists(path);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    });
}
```

---

# 57. Recommended Download Validation

A strong download test should validate:

```text
1. Download action is available
2. Download event occurs
3. Download does not fail
4. Expected file name
5. Expected extension
6. File exists
7. File size is greater than zero
8. File content is correct
```

Example:

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);

Assert.assertNull(download.failure());

String fileName =
    download.suggestedFilename();

Assert.assertTrue(
    fileName.endsWith(".pdf")
);

Path path =
    Paths.get("target/downloads")
         .resolve(fileName);

download.saveAs(path);

Assert.assertTrue(
    Files.exists(path)
);

Assert.assertTrue(
    Files.size(path) > 0
);
```

---

# 58. Common Mistakes

## Mistake 1: Using Selenium-style OS automation

Avoid using tools such as:

```text
Robot
AutoIT
Sikuli
Windows file chooser automation
```

when Playwright's file APIs can handle the operation directly.

---

## Mistake 2: Clicking Upload Before Waiting for File Chooser

Incorrect:

```java
page.getByText("Upload").click();

FileChooser chooser =
    page.waitForFileChooser(...);
```

The event may already have happened.

Correct:

```java
FileChooser chooser =
    page.waitForFileChooser(
        () -> page.getByText("Upload").click()
    );

chooser.setFiles(file);
```

---

## Mistake 3: Not Waiting for Download

Avoid:

```java
page.getByText("Download").click();
```

and immediately assuming a file exists.

Use:

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);
```

---

## Mistake 4: Hardcoding Temporary Browser Download Paths

Avoid relying on browser-specific temporary paths.

Prefer:

```java
download.saveAs(
    Paths.get("target/downloads/report.pdf")
);
```

---

## Mistake 5: Only Checking the File Name

A file can have the correct name but still be empty or corrupted.

Also validate:

```java
Files.exists(path)
```

and:

```java
Files.size(path) > 0
```

For important files, validate actual content as well.

---

# 59. Selenium vs Playwright File Upload

### Selenium

Selenium commonly uses:

```java
driver.findElement(By.id("upload"))
      .sendKeys("/path/to/file.pdf");
```

### Playwright

Playwright uses:

```java
page.locator("#upload")
    .setInputFiles(
        Paths.get("path/to/file.pdf")
    );
```

Playwright also provides:

```java
FileChooser
```

for file chooser workflows.

---

# 60. Selenium vs Playwright File Download

Selenium often requires browser preferences or external download configuration.

Playwright provides:

```java
Download download = page.waitForDownload(
    () -> page.getByText("Download").click()
);
```

Then:

```java
download.saveAs(path);
```

This makes download testing straightforward and deterministic.

---

# 61. Interview Questions

## Q1. How do you upload a file in Playwright?

```java
page.locator("input[type='file']")
    .setInputFiles(
        Paths.get("src/test/resources/test.pdf")
    );
```

---

## Q2. How do you upload multiple files?

```java
page.locator("input[type='file']")
    .setInputFiles(
        Paths.get("file1.pdf"),
        Paths.get("file2.pdf")
    );
```

---

## Q3. How do you handle a file chooser?

```java
FileChooser chooser =
    page.waitForFileChooser(
        () -> page.getByText("Upload").click()
    );

chooser.setFiles(file);
```

---

## Q4. How do you handle downloads?

```java
Download download =
    page.waitForDownload(
        () -> page.getByText("Download").click()
    );
```

---

## Q5. How do you save a downloaded file?

```java
download.saveAs(
    Paths.get("target/downloads/file.pdf")
);
```

---

## Q6. How do you get the downloaded file name?

```java
String fileName =
    download.suggestedFilename();
```

---

## Q7. How do you verify a download?

```java
Path path =
    Paths.get("target/downloads/file.pdf");

download.saveAs(path);

Assert.assertTrue(
    Files.exists(path)
);

Assert.assertTrue(
    Files.size(path) > 0
);
```

---

## Q8. How do you check whether the download failed?

```java
String failure = download.failure();

Assert.assertNull(failure);
```

---

## Q9. Can Playwright upload multiple files?

Yes.

```java
page.locator("input[type='file']")
    .setInputFiles(file1, file2, file3);
```

---

## Q10. Can Playwright handle downloads in headless mode?

Yes. Playwright's download API works in both headed and headless browser execution.

---

## Q11. Do you need AutoIT for file uploads in Playwright?

Usually no.

Playwright provides:

```java
setInputFiles()
```

and:

```java
FileChooser
```

for file uploads.

---

## Q12. What is the difference between `setInputFiles()` and `FileChooser`?

`setInputFiles()` is normally used when the file input element is directly accessible.

```java
page.locator("input[type='file']")
    .setInputFiles(file);
```

`FileChooser` is useful when clicking an application control opens the file chooser.

```java
FileChooser chooser =
    page.waitForFileChooser(
        () -> page.getByText("Upload").click()
    );

chooser.setFiles(file);
```

---

# 62. Best Practices

1. Prefer `setInputFiles()` for normal file inputs.
2. Use `waitForFileChooser()` for file chooser events.
3. Use `waitForDownload()` for downloads.
4. Always save downloads to a controlled test directory.
5. Validate the downloaded file.
6. Verify file size.
7. Validate file content for important downloads.
8. Keep test files under `src/test/resources`.
9. Avoid hardcoded developer-specific absolute paths.
10. Use Page Object Model for reusable upload/download operations.
11. Clean the download directory before tests when necessary.
12. Test both valid and invalid file types.
13. Test maximum file size.
14. Test multiple-file uploads where applicable.
15. Test missing-file validation.
16. Avoid OS-level automation when Playwright APIs can handle the workflow.
17. Make download file names dynamic when the application generates them.
18. Keep upload/download utilities reusable across test classes.

---

# 63. Recommended Framework Structure

```text
src
└── test
    ├── java
    │   ├── pages
    │   │   ├── UploadPage.java
    │   │   └── DownloadPage.java
    │   │
    │   ├── tests
    │   │   ├── FileUploadTest.java
    │   │   └── FileDownloadTest.java
    │   │
    │   └── utils
    │       └── FileUtils.java
    │
    └── resources
        └── testdata
            ├── test.pdf
            ├── test.csv
            ├── test.txt
            └── test.png

target
└── downloads
```

---

# 64. End-to-End Example

```java
import com.microsoft.playwright.*;
import org.junit.jupiter.api.Test;

import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

import static org.junit.jupiter.api.Assertions.*;

public class FileUploadDownloadTest {

    @Test
    public void uploadAndDownloadTest() throws Exception {

        try (Playwright playwright = Playwright.create()) {

            Browser browser =
                playwright.chromium().launch(
                    new BrowserType.LaunchOptions()
                        .setHeadless(false)
                );

            Page page = browser.newPage();

            page.navigate(
                "https://example.com/file-management"
            );

            // Upload
            Path uploadFile =
                Paths.get(
                    "src/test/resources/testdata/test.pdf"
                );

            page.locator("input[type='file']")
                .setInputFiles(uploadFile);

            page.getByRole(
                AriaRole.BUTTON,
                new Page.GetByRoleOptions()
                    .setName("Upload")
            ).click();

            assertTrue(
                page.getByText(
                    "File uploaded successfully"
                ).isVisible()
            );

            // Download
            Download download =
                page.waitForDownload(
                    () -> page.getByText("Download").click()
                );

            assertNull(download.failure());

            String fileName =
                download.suggestedFilename();

            assertTrue(
                fileName.endsWith(".pdf")
            );

            Path downloadDirectory =
                Paths.get("target/downloads");

            Files.createDirectories(downloadDirectory);

            Path downloadedFile =
                downloadDirectory.resolve(fileName);

            download.saveAs(downloadedFile);

            // Validate
            assertTrue(
                Files.exists(downloadedFile)
            );

            assertTrue(
                Files.size(downloadedFile) > 0
            );

            browser.close();
        }
    }
}
```

---

# 65. Quick Reference

| Operation              | Playwright Java               |
| ---------------------- | ----------------------------- |
| Upload file            | `setInputFiles()`             |
| Upload multiple files  | `setInputFiles(file1, file2)` |
| File chooser           | `waitForFileChooser()`        |
| Set chooser files      | `chooser.setFiles()`          |
| Clear selected files   | `setInputFiles(new Path[0])`  |
| Wait for download      | `waitForDownload()`           |
| Get file name          | `suggestedFilename()`         |
| Save download          | `saveAs()`                    |
| Get temporary path     | `path()`                      |
| Check download failure | `failure()`                   |
| Verify file exists     | `Files.exists()`              |
| Verify file size       | `Files.size()`                |
| Read text file         | `Files.readString()`          |

---

# 66. Key Takeaways

```text
Upload:
page.locator("input[type='file']")
    .setInputFiles(file);

File Chooser:
FileChooser chooser =
    page.waitForFileChooser(
        () -> page.getByText("Upload").click()
    );

chooser.setFiles(file);

Download:
Download download =
    page.waitForDownload(
        () -> page.getByText("Download").click()
    );

Save:
download.saveAs(path);

File Name:
download.suggestedFilename();

Failure:
download.failure();

Existence:
Files.exists(path);

Size:
Files.size(path);
```

Playwright makes file upload and download automation much simpler than traditional browser automation because file operations are built directly into the framework. The recommended approach is to use Playwright's `setInputFiles()`, `FileChooser`, and `Download` APIs instead of relying on operating-system-level automation.
