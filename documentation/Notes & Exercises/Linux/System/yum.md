​        

## Undoing Installs

If you install a package with `yum install`, say `pdftk`, it will pull in a lot of dependencies:

```
Installed:
  pdftk.x86_64 0:1.44-10.fc18

Dependency Installed:
  bouncycastle.noarch 0:1.46-6.fc18     
  itext-core.noarch 0:2.1.7-14.fc18     
  libgcj.x86_64 0:4.7.2-8.fc18          
  bouncycastle-mail.noarch 0:1.46-6.fc18
  java-1.5.0-gcj.x86_64 0:1.5.0.0-40.fc18
  sinjdoc.x86_64 0:0.5-13.fc18
  bouncycastle-tsp.noarch 0:1.46-5.fc18
  java_cup.noarch 1:0.11a-10.fc18
  itext.x86_64 0:2.1.7-14.fc18   
  javamail.noarch 0:1.4.3-12.fc18

Complete!
```

`yum remove pdftk` will remove only that package and not all the dependencies.

But you can look at all the 'transactions' (install, remove etc.):

```
$ sudo yum history list pdftk
ID     | Command line             | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------  
    88 | install pdftk            | 2012-12-14 13:35 | Install        |   11   
```

And then you can undo that transaction:

```
$ sudo yum history undo 88
Undoing transaction 88, from Fri Dec 14 13:35:34 2012
    Dep-Install bouncycastle-1.46-6.fc18.noarch       @fedora
    Dep-Install bouncycastle-mail-1.46-6.fc18.noarch  @fedora
    Dep-Install bouncycastle-tsp-1.46-5.fc18.noarch   @fedora
    Dep-Install itext-2.1.7-14.fc18.x86_64            @fedora
    Dep-Install itext-core-2.1.7-14.fc18.noarch       @fedora
    Dep-Install java-1.5.0-gcj-1.5.0.0-40.fc18.x86_64 @fedora
    Dep-Install java_cup-1:0.11a-10.fc18.noarch       @fedora
    Dep-Install javamail-1.4.3-12.fc18.noarch         @fedora
    Dep-Install libgcj-4.7.2-8.fc18.x86_64            @fedora
    Install     pdftk-1.44-10.fc18.x86_64             @fedora
    Dep-Install sinjdoc-0.5-13.fc18.x86_64            @fedora
    ...
    Complete!
```