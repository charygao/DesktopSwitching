# Desktop Switching
Fork from [CodeProject](http://www.codeproject.com/Articles/7666/Desktop-Switching "By Nnamdi Onyeyiri")

![download](https://cloud.githubusercontent.com/assets/2945229/14548941/5f787dd2-0270-11e6-9d30-622ccdeaf302.png) [Download source - 6 Kb](http://www.codeproject.com/KB/cs/CsDesktopSwitching/Desktop_src.zip)

# Introduction

Having a spare computer available, I decided to try out Mandrake Linux, just to see what its like (having never used linux before). I thought its desktop switching ability was interesting, and began wondering if it was possible to do the same thing on my Windows box. A quick search on MSDN revealed the Desktop API functions, which allowed me to do just that. According to the docs, this functionality has been available since Windows 2000 Professional, and since I'm using XP, I thought I would give it a go.

When Windows loads, and you are logged in, a desktop is automatically created, called "default". Using the functions provided you can create, open, and switch desktops, unfortunately, deletion isn't provided (more on that later).

# Using the Desktop class

**Creating a Desktop**

Creating a desktop is a very easy task, as the code example below shows. Desktops can be created using either the instance, or the static members provided.

```csharp
// Instance method
Desktop desktop = new Desktop();
desktop.Create("myDesktop");
 
// Static method
Desktop desktop = Desktop.CreateDesktop("myDesktop");
```
    
**Note:** Desktop names cannot contain backslash characters.

Using either of the methods will result in a Desktop object, with an open handle to the desktop (provided the desktop was created successfully). New desktops have no processes running on them at all - and if you were to switch to it, all you would see is the default wallpaper. I have provided the Prepare method which loads explorer to the desktop, making it usable.

**Switching Desktops**

Switching desktops is just as easy as creating the desktop, all that is required is a call to either the static method stating the desktop name, or the instance method of an open Desktop object.

```csharp
// instance method.
Desktop desktop = new Desktop();
desktop.Open("myDesktop");
desktop.Show();
 
// static method.
Desktop.Show("myDesktop");
```

**Opening Processes in Desktops**

The CreateProcess function in kernel32.dll takes a parameter of type STARTUPINFO, which has a parameter ("lpDesktop") which allows you to specify the desktop the process is to be created in. As a result, I chose to import this function, instead of using the .NET frameworks Process class, meaning reduced functionality when creating process, but I hope to resolve this in the next release. Now the code, this example shows the two ways of creating a process.

```csharp
// instance method
Desktop desktop = Desktop.OpenDesktop("myDesktop");
Process p = desktop.CreateProcess("calc.exe");
 
// static method
Process p = Desktop.CreateProcess("calc.exe", "myDesktop");
```

A Process object is returned from both CreateProcess methods, which can be used for killing the process, or however you see fit.

**Deleting Desktops**

Deleting a desktop is a little trickier. The only way to delete a desktop is to kill all processes running on it, at which point, it is automatically deleted. So far, I have been unable to get a list of processes running on a desktop other than the input desktop (nor am I certain this is possible), but what I have done, is provided the GetInputProcesses method, which will return an array of all the processes running on the current input desktop (the desktop visible to the user).

With this in mind, the only way to delete a desktop which has been accessed by the user is to be on the desktop, enumerate the processes, and kill them one-by-one. Making sure your application isn't killed off before it has a chance to jump desktops (unless you want it that way).

```csharp
Process[] processes = Desktop.GetInputProcesses();
Process thisProc = Process.GetCurrentProcess();
foreach(Process p in processes)
{
   if (p.ProcessName != thisProc.ProcessName)
   {
      p.Kill();
   }
}
```

**Settings a Thread's Desktop**

Threads of your process can be moved between desktops, provided they do not have any hooks or windows in the current desktop.

```csharp
Desktop desktop = Desktop.OpenDesktop("myDesktop");
Desktop.SetCurrent(desktop);
```
