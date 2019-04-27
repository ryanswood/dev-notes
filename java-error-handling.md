Java:

Errors

Unchecked vs checked:

1) Checked: are the exceptions that are checked at compile time. If some code within a method throws a checked exception, then the method must either handle the exception or it must specify the exception using throws keyword.

2) Unchecked are the exceptions that are not checked at compiled time. In C++, all exceptions are unchecked, so it is not forced by the compiler to either handle or specify the exception. It is up to the programmers to be civilized, and specify or catch the exceptions.
In Java exceptions under Error and RuntimeException classes are unchecked exceptions, everything else under throwable is checked.

￼

Oracle’s suggestion:
Should we make our exceptions checked or unchecked?
Following is the bottom line from Java documents
If a client can reasonably be expected to recover from an exception, make it a checked exception. If a client cannot do anything to recover from the exception, make it an unchecked exception

Resources:
https://www.geeksforgeeks.org/checked-vs-unchecked-exceptions-in-java/
https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html
https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-exceptions.html


try-with-resouces

Is an easy way to declare dependencies that should be closed at the end of the method. The issue is that the dependencies may raise an IOException that must be handled if the close operation fails. This means that we can’t differentiate between IOExceptions that occur in the main method logic or in the close operation. We can safely ignore the failed close but not the logic exception.

https://stackoverflow.com/questions/48705633/how-should-i-use-ioexception-in-try-with-resources-nested-in-the-method-witht
http://tutorials.jenkov.com/java-exception-handling/try-with-resources.html


try-catch-finally
A method that uses a try block and expects a value to be returned must have all try, catch, finally blocks return the expected type or raise an exception. To get around empty catch and finally blocks, initialize a return variable outside of the try scope and and the end of the try block return the value. The try block should assign a value.

```
int num = null;
try {
  num = 1;
} catch {
    // Log something
} finally {
  // Do some cleanup
}

return num;
```

What should happen in a catch block:

https://www.codejava.net/java-core/exception/5-rules-about-catching-exceptions-in-java

Options for automatically deleting files:
- https://softwarecave.org/2014/02/05/create-temporary-files-and-directories-using-java-nio2/

FIle vs Files package:
https://docs.oracle.com/javase/tutorial/essential/io/legacy.html



S3:
Reading an S3 Object in Java:
https://docs.aws.amazon.com/AmazonS3/latest/dev/RetrievingObjectUsingJava.html
