# Java

## Install
* Use openjdk provided as an archive file at [https://jdk.java.net/](https://jdk.java.net/)
* unzip in a directory of your choice
* define JAVA_HOME environment variable
  ** update your shell profile `export JAVA_HOME=mydirectory`
  ** on windows, open a command prompt and run `setx JAVA_HOME mydirectory`
* update your PATH to include JAVA_HOME/bin
  ** update your shell profile `export PATH=$PATH:$JAVA_HOME/bin`
  ** on windows, open a command prompt and run `setx PATH mydirectory` (mind any previous definition will be overriden)

## Test
* open a command shell
* simply run `java -version`
* ensure version displayed is the one you installed !
 
