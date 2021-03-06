# Rsession: R sessions wrapping for Java #

Rsession provides an easy to use java class giving access to remote or local R session. The back-end engine is Rserve, locally spawned automatically if necessary.

Rsession differs from Rserve as it is a higher level API, and it includes server side startup of Rserve. Therefore, it is easier to use in some point of vue, as it provides a multi session R engine (including for Windows, thanks to an ugly turn-around).

Another alternative is JRI, but it does not provide multi-sessions feature. If you just need one R session in your java code, JRI is a good solution.
## Example Java code ##
```java
import static org.math.R.Rsession.*;
...
 
    public static void main(String args[]) {
        Rsession s = Rsession.newInstanceTry(System.out);
 
        double[] rand = (double[]) s.eval("rnorm(10)",null); //create java variable from R command
 
        ...
 
        s.set("c", Math.random()); //create R variable from java one
 
        s.save(new File("save.Rdata"), "c"); //save variables in .Rdata
        s.rm("c"); //delete variable in R environment
        s.load(new File("save.Rdata")); //load R variable from .Rdata
 
        ...
 
        s.set("df", new double[][]{{1, 2, 3}, {4, 5, 6}, {7, 8, 9}, {10, 11, 12}}, "x1", "x2", "x3"); //create data frame from given vectors
        double value= (Double) (cast(s.eval("df$x1[3]"))); //access one value in data frame
 
        ...
 
        s.toJPEG(new File("plot.jpg"), 400, 400, "plot(rnorm(10))"); //create jpeg file from R graphical command (like plot)
 
        String html = s.asHTML("summary(rnorm(100))"); //format in html using R2HTML
        System.out.println(html);
 
        String txt = s.asString("summary(rnorm(100))"); //format in text
        System.out.println(txt);
 
        ...
 
        System.out.println(s.installPackage("sensitivity", true)); //install and load R package
        System.out.println(s.installPackage("wavelets", true));
 
        s.end();
    }
```
## Use it ##

* install R from http://cran.r-project.org
* get Rsession libs:
  * (hard) build it yourself:
            checkout Rsession
            build Rsession: (command line inside Rsession dir) ant dist
            copy 'dist/lib' directory inside your java project (as 'lib' directory) 
  * (easy) or just unzip prebuilt archive: https://github.com/yannrichet/rsession/blob/master/Rsession/dist/libRsession.zip in your 'lib' directory 
* add lib/Rsession*.jar:lib/Rserve*.jar:lib/REngine*.jar in your project classpath
* use in your code:
  * create new Rsession:
  * (easy) local spawning of Rserve (for Windows XP, Mac OS X, Linux 32 & 64):
    ```java
    Rsession s = Rsession.newInstanceTry(System.out,null);
    ```
  * OR connect to remote Rserve (previously started with /usr/bin/R CMD Rserve --vanilla --RS-conf Rserve.conf):
    ```java
    Rsession s = Rsession.newRemoteInstance(System.out,RserverConf.parse("R://192.168.1.1"));
    //connect to local Rserve (previously started with /usr/bin/R CMD Rserve --vanilla --RS-conf Rserve.conf):
    Rsession s = Rsession.newLocalInstance(System.out,null); 
    ```
  * do your work in R and get Java objects
    * create Java objects from R command using
    ```java
    Object o = s.eval("...",HashMap<String,Object> vars)
    ```
    (Object o is automatically cast to double, double, double,String, String, ...)
    * OR
      create your R objects using `s.set("...",...)`
      call any R command using `s.evalR("...")`
      cast to Java objects using `Rsession.Rcast(...)`
      if needed use remote R packages install & load: `s.installPackage("...", true);`
      you can access R command answers as string using: `s.asHTML("...")` `s.asString("...")` , `s.toJPEG(File f,"...")` 
  * finally close your Rserve instance: `s.end(); `

