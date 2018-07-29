# I/O and NIO

### I/O classes
- File
- FileReader
- BufferedReader
- FileWriter
- BufferedWriter
- PrintWriter
- FileInputStream
- FileOutputStream
- ObjectInputStream
- ObjectOutputStream

:fireworks: Classes with `Stream` in their name are used to read and write bytes, and `Readers` and `Writers` are used to read and write characters.

- #### File
```Java
import java.io.File;

public class Writer1 {
    public static void main(String[] args) {
        File file = new File("myfile.txt"); //there is no file yet
    }
}
```
If you compile and run the program, you will not find any file called `myfile.txt`. When you make a new instance of the class *File, you are not making an actual file; you are just creating a file name.*

We extend the same example above

```Java
import java.io.File;
import java.io.IOException;

public class Writer1 {
    public static void main(String[] args) throws IOException {
        File file = new File("myfile.txt"); //there is no file yet
        System.out.println(file.exists()); //looks for a real file

        boolean newFile = file.createNewFile(); //create a file if not exists
        System.out.println(newFile);

        System.out.println(file.exists());
    }
}
```

This produces output as
- *false*
- *true*
- *true*

:fireworks:
- **boolean exists()** : return true if it can find an actual file.
- **boolean createNewFile()** : creates new file if it doesn't exists.

- #### Working with File and Directories
Create a `Writer` or `Stream`, specially create a `FileWriter`, a `PrintWriter` or a `FileOutputStream`. When you create an instance of one of these classes, you automatically create a file, unless one already exists.

```Java
File file2 = new File("foo.txt"); // no file yet
PrintWriter writer = new PrintWriter(file2); //makes a printwriter object and make a file.
```

But that is not the same case when you have to create a file under a directory.
i.e.
```Java
File mydir = new File("mydir");
File file2 = new File(mydir, "foo.txt");

file2.createNewFile(); //exception if no directory exists.
```
This will generate an exception;
```console
java.io.IOException: No such file or directory
```
For this to work we have to call *mydir.mkDir()*
```Java
File mydir = new File("mydir");
mydir.mkdir();
File file2 = new File(mydir, "foo.txt");

file2.createNewFile();
```

### Files, Path and Paths
Objective: Operate on file and directory paths by using the Paths class.

- Path : This is an interface which replaces `File` as the representation of a file or directory while working with NIO2.
- Paths : This class contains static methods that create `Path` objects by implementing `Path` interface.
- Files : This class contains static methods that work with `Path` objects. Uses `Path` object as parameter.
- File : it's been around since the beginning. `File` and `Path` objects know how to convert each other.

![Alt Files Path Relationship](../images/filespathrel.png)

:fireworks: `Files` and `File` despite similarity in names, these objects don't know each other.

```Java
Path p1 = Paths.get("foo/file1.txt");
Path p2 = Paths.get("foo", "file1.txt");
//p1 and p2 are same

System.out.println(Files.exists(p1));

Path dir = Paths.get("foo");
Files.createDirectories(dir); //create directories if not exists

Files.createFile(p1);
System.out.println(Files.exists(p1));
```
### I/O vs NIO2

| Description   | I/O           | NIO2  |
| ------------- |:-------------| :----- |
| Create an empty file.      | File f = new File("test.txt"); f.createNewFile(); | Path path = Paths.get("test.txt"); Files.createFile(path); |
| Create a new Directory     | File f = new File("dir"); f.mkDir();      |   Path path = Paths.get("dir"); Files.createDirectory(path); |
| Create directories including any missing parent directories | File f = new File("a/b/c"); f.mkdirs();      |    Path path = Paths.get("a/b/c"); Files.createDirectories(path); |
| Check if a file or directory exists | File f = new File("test.txt"); f.exists();      |    Path path = Paths.get("test.txt"); Files.exists(path); |

### Copying, Moving and Deleting files.
```Java
Path one = Paths.get("foo/file1.txt"); //exists
Path two = Paths.get("foo/file2.txt"); //exists

Path targ = Paths.get("foo/file12.txt"); //doesn't yet exist
Files.copy(one, targ);
Files.copy(two, targ); //oops, java.nio.file.FileAlreadyExistsException: foo/file12.txt
```
When Java see it's about to override the file, it throws _FileAlreadyExistsException_ as we haven't provided any option incase the file is about to replaced.

```Java
Files.copy(two, targ, StandardCopyOption.REPLACE_EXISTING);
```
Similarly, while trying to delete a file which doesn't exist, `NoSuchFileException` is thrown.
We can safeguard this by using
```Java
Files.deleteIfExists(path);
```
### Retrieving information about a Path
The Path interface defines a bunch of methods that return useful information about the path.

```Java
Path path = Paths.get("/home/java/workspace.note");
System.out.println("getFaileName(): " + path.getFileName());
System.out.println("getName(1): " + path.getName(1));
System.out.println(("getNameCount(): "+ path.getNameCount()));
System.out.println("getParent(): "+path.getParent());
System.out.println("getRoot(): "+path.getRoot());
System.out.println(("subpath(0,2): " + path.subpath(0, 2)));
System.out.println(("toString()" + path.toString()));
```
This produces the following output.
```console
getFaileName(): workspace.note
getName(1): java
getNameCount(): 3
getParent(): /home/java
getRoot(): /
subpath(0,2): home/java
toString()/home/java/workspace.note
```
Another interesting fact about `Path` interface is it extends from `Iterable<Path>`, hence we can iterate with enhance `for` loop.

```Java
int spaces = 1;
Path myPath = Paths.get("tmp", "dir1", "dir2", "dir3", "file.txt");
for (Path subPath : myPath) {
    System.out.format("%" + spaces + "s%s%n", "", subPath);
    spaces +=2;
}
```
which produces the following output:
```console
tmp
  dir1
    dir2
      dir3
        file.txt
```
### Resolving a Path (combining two `Path`)
```Java
Path dir = Paths.get("/home/java");
Path file = Paths.get("models/Model.java");
Path result = dir.resolve(file);
System.out.println("Result: " + result);
```
This produces the absolute path by merging two paths.
```console
Result: /home/java/models/Model.java
```

:fireworks: `resolve` will not check that the directory or file actually exists.

:tada: **Exam Note:**
resolve() methods comes in two falvours. One with a `Path` parameter and the other with a `String` parameter. The tricky part here is that null is avalid value for both a `Path` and `String`. What will happen if you pass `null` as a parameter ?

![Resolve Path](../images/resolvepath.png)

The compiler can't decide which method to invoke; hence the code **won't compile**.

The following example will _compile without any problem_; because the compiler knows which method to invoke.
```java
Path path = Paths.get("/use/bin");
Path other = null;
path.resolve(other);

path.resolve((String) null);
```
### Relativizing a Path
`Relativize` is just opposite of `resolve`.
`path1.relativize(path2)` means _**give me a path that shows how to go path2 from path1**_

```Java
Path dir = Paths.get("/home/java");
Path music = Paths.get("/home/java/country/gana.mp3");
Path mp3 = dir.relativize(music);
System.out.println(mp3);
```
The output is
```console
country/gana.mp3
```
Now we can try some complex example:
```Java
Path absolute1 = Paths.get("/home/java");
Path absolute2 = Paths.get("/usr/local");
Path absolute3 = Paths.get("/home/java/temp/music.mp3");
Path relative1 = Paths.get("temp");
Path relative2 = Paths.get("temp/music.pdf");
System.out.println("1: "+ absolute1.relativize(absolute3));
System.out.println("2: "+ absolute3.relativize(absolute1));
System.out.println("3: "+ absolute1.relativize(absolute2));
System.out.println("4: "+ relative1.relativize(relative2));
```
The output is
```console
1: temp/music.mp3
2: ../..
3: ../../usr/local
4: music.pdf
```
:tada: Remember `relativize()` and `resolve()` are opposite. Just like `resolve()`, `relativize()` does not check that the path actually exists.

### DirectoryStream
DirectoryStream works like `dir` command in DOS and the `ls` command in UNIX. It can look only one directory. i.e. we have the following directory.
/home
  | - users
          |  -  vafi
          |  -  eyra


```Java
Path dir = Paths.get("/home/users");
try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir)) {
    //try-with-resources, so we don't have to close the stream explicitly.
    for (Path path : stream) {
        System.out.println(path);
    }
}
```
The output is
```console
vafi
eyra
```
Suppose we want to know the name of directory whose name starts with `v` or `w`.
- "v" or "w" followed  by anything
```java
Path dir = Paths.get("/home/users");
try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir, "[vw]*")) {
    //try-with-resources, so we don't have to close the stream explicitly.
    for (Path path : stream) {
        System.out.println(path);
    }
}
```
This time the output is
```console
vafi
```
:tada: The `*` is a wildcard that means ZERO or more of any characters. Notice this is not a regular expression. DirectoryStream uses something called `glob`.

### FileVisitor
