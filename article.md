# VS 2017 offline installation folder cleanup

## Introduction

Some times you have to wonder what the folks at Microsoft are thinking. If you have tried the offline installation mode with the new Visual Studio 2017 installation mechanism, you will see that on subsequent update download your offline folder size will increase, that is to say the old versions are not deleted and the full install 28Gb directory will just get bigger.

To help relieve you of the burden on disk, I have written a small program that will cleanup that folder (it should be in the installer but it's not). 

## How to use it

The usage is pretty simple, just copy the EXE and the DLL file into the offline installation folder and run from there, and you will see the output like below:

[pic]

## What's in the code

The code is very simple, it reads the `catalog.json` file and goes through all the folders of VS2017 and tries to match the package `id` and `version` number, if non is found it presumes it is an old version and moves that folder to the `- old - ` folder where you delete later.

```c#
class Program
{
	static void Main(string[] args)
	{
		string path = Directory.GetCurrentDirectory();
		if (path.EndsWith("\\") == false) path += "\\";
		string catalog = path + "catalog.json";
		if (File.Exists(catalog) == false)
		{
			Console.WriteLine("Please copy this program to the VS2017 offline folder.");
			return;
		}
		string old = Directory.CreateDirectory(path + "- old -").FullName;
		var o = fastJSON.JSON.ToDynamic(File.ReadAllText(catalog));

		var folders = Directory.GetDirectories(path);
		var packages = o["packages"];
		foreach (var folder in folders)
		{
			var foldername = Path.GetFileName(folder);
			var splitstring = foldername.Split(',');
			bool found = false;
			if (splitstring.Length > 1)
			{
				foreach (var package in packages)
				{
					if (package["id"].ToString() == splitstring[0] && package["version"] == splitstring[1].Split('=')[1].Trim())
					{
						found = true;
						break;
					}
				}
			}
			else
			{
				found = true;
				Console.WriteLine("not found in packages : " + foldername);
			}

			if (found == false)
			{
				// move directory
				Console.WriteLine("moving folder : " + foldername);
				Directory.Move(folder, old + "\\" + foldername);
			}
		}
		Console.WriteLine("Press any key to exit.");
		Console.ReadKey();
	}
} 
```



## Points of Interest

I'm using my `fastJSON` library to parse the `catalog.json` file in the installation folder and using `ToDynamic()` to simplify the coding.

## History

