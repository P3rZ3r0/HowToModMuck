# HowToModMuck
How to mod Muck (Dani's latest released game)

# Requirements
Before you mod muck or any other unity games you need to have same basic programming knowledge. Since Unity and Bepinex use C#, we're going to be using C#
You will need an IDE. I recommend these 2: https://www.jetbrains.com/rider/ or https://visualstudio.microsoft.com/
Rider has a free 30 day trial and Visual Studio is free (I use and recommend Rider because of its code completion and a bunch of extra features)
Next we'll need a decompiler. https://github.com/dnSpy/dnSpy https://www.jetbrains.com/decompiler/ https://github.com/icsharpcode/AvaloniaILSpy
Also make sure that you have https://dotnet.microsoft.com/download/dotnet-framework/net48 (the Runtime and the dev pack)

# Mod Loader
There is a couple of mod loaders for Unity games. We'll be using Bepinex. https://bepis.io/unity.html - comparison between mod loaders
Let's start by downloading Bepinex
https://github.com/BepInEx/BepInEx/releases get the latest, 64bit release from the releases

To install bepinex go on steam and right click Muck
Click on properties and then local files
Click browse, this should take you to the Muck folder
Inside the muck folder extract everything from the bepinex zip and run the game

# Configuring Bepinex
We'll need the console for developing mods, this will help us debug and fix errors
Inside the Muck folder go to the Bepinex folder -> config -> BePinEx.cfg and open it up in your editor of choice
Scroll down to the Logging.Console section and inside there set Enabled from false to true
Save the file and run the game, make a console popped up

# Handy mods for development

https://docs.bepinex.dev/v5.4.4/articles/dev_guide/dev_tools.html

You should read through that. You'll for sure need the Script Engine and Runtime Unity Editor
https://github.com/BepInEx/BepInEx.Debug/releases/download/r7/ScriptEngine_v7.0.zip
https://github.com/ManlyMarco/RuntimeUnityEditor/releases/download/v2.4/RuntimeUnityEditor_BepInEx5_v2.4.zip

You reload scripts by pressing F6 and open up the unity editor by pressing F12 (keep in mind this will cause huge lag, you should decrease your graphics settings)

To install these mods you put them in the plugins folder in the Bepinex folder

# Creating your first mod

Please keep in mind I am going to be using rider in this section, if you're going to be using visual studio it may be different

First we'll create a solution

![image](https://user-images.githubusercontent.com/61652884/121035735-753b7580-c7ae-11eb-83fe-334b5064234a.png)

We'll choose class library, make sure you setup the right framework (should be v4.8 if you installed it from the requirements section)

![image](https://user-images.githubusercontent.com/61652884/121035964-ab78f500-c7ae-11eb-8f91-c347a40f9af2.png)

First we'll import all of the dependencies that are required for modding
In Rider you can right click on the folder in the hierarchy and click show in explorer

![image](https://user-images.githubusercontent.com/61652884/121036781-39ed7680-c7af-11eb-898c-f06473f4b596.png)

Create a folder and name it libs

After that go inside Muck's folder then Bepinex -> core
select Bepinex.dll Bepinex.Harmony.dll and Harmony0.dll copy them and paste them inside the Libs folder
Go back inside the muck folder and then go Muck_Data -> Managed and select Assembly-CShrap.dll, UnityEngine.dll and UnityEngine.CoreModule.dll, copy those and paste them in the libs folder
Please keep in mind you will need some other unity libs depending on what're you doing, if you for example want to raycast you will need the UnityEngine.PhysicsModule.dll
You should be able to find which dll you need for what thing on unity's documentation
![image](https://user-images.githubusercontent.com/61652884/121037730-f9dac380-c7af-11eb-8069-bbc8603e4c53.png)

![image](https://user-images.githubusercontent.com/61652884/121037837-14ad3800-c7b0-11eb-93b3-06e3678d189d.png)

This is how your libs folder should look like right now

Now go back to your IDE and let's import these libs, on Rider you can open up the entire hierarchy for your project and right click Dependencies then click add reference -> Add from -> Go inside the libs folder and select everything from there and click open

Now you should have all of the dependencies added in your project

First let's start by renaming our class from Class1 to MainClass (or something similar) this step isn't required but it will make your project look more professional

Now open up the class and before the line with public class MainClass add [BepInPlugin("GUID", "PluginName", "PluginVersion")]
Change GUID to something like me.yourname.pluginname, change pluginname to your mod's name and change PluginVersion to this format: 1.0.0.0
The first number is a major release, the second number is a minor release, the third number is a build and the fourth number is fix/revision

Then add : BaseUnityPlugin after public class MainClass

![image](https://user-images.githubusercontent.com/61652884/121039315-4541a180-c7b1-11eb-8011-000736ef2ca8.png)

now lets add some variables

First we'll add the instance variable this is how the line will look:
public static MainClass instance;

We'll be using this instance to access some variables that are going to be inside the MainClass
Then we add a logger for debuging info and writing into the console:
public ManualLogSource log

and the last variable is going to be harmony
public Harmony harmony

Now we create our first method

private void Awake() {

}

inside this method we'll first define the instance, this is how we do it

if (instance == null)
                instance = this;
            else
                Destroy(this);
                
then we define the log variable
log = Logger

and the last one is harmony which we define like this:

harmony = new Harmony("GUID")

You should already know what to replace GUID with

![image](https://user-images.githubusercontent.com/61652884/121041032-addd4e00-c7b2-11eb-9b04-816859ccc93d.png)

We're done with the method for now

Patching our first method

Finally we're going to be making something that will change something about the game for this we'll need a decompiler
Open up a decompiler of choice, grab the Assembly-CSharp.dll file and drag it into the decompiler, this should open it up in the decompiler

Dnspy:
![image](https://user-images.githubusercontent.com/61652884/121042233-c4d07000-c7b3-11eb-8acc-302fb7322f77.png)

Dotpeek:
![image](https://user-images.githubusercontent.com/61652884/121042305-d580e600-c7b3-11eb-9ce3-b89663fa3522.png)

You can scroll through all of the classes
As an example I am going to create a mod that will change our readyToJump variable in the PlayerMovement class always true

First we'll create another class inside the MainClass file like this:

![image](https://user-images.githubusercontent.com/61652884/121043219-b8004c00-c7b4-11eb-9ff3-c7cd449ea76a.png)

We'll first need to tell what class and which method we want to patch which we'll do like this

[HarmonyPatch(typeof(PlayerMovement), "Movement")]

PlayerMovement is the class and the string Movement is the method

Then we'll define if we want to patch before the method is ran or after, we want before it is ran

[HarmonyPrefix]

if you wanna do something after the method ran you'll do this

[HarmonyPostfix]

Since we're using prefix we'll need our method to return a boolean so this is how we create the method

static bool PrefixMovement()

since we want to change readyToJump to true we'll need to define it, readyToJump is a global variable so we put 3 underscores before it like this

static bool PrefixMovement(bool ___readyToJump)

but we want to edit this value so we put ref behind it like this

static bool PrefixMovement(ref bool ___readyToJump)

and now we just set readytojump to true like this

___readyToJump = true

and end the method like this

return true

returning true will not return out of the method thus the rest of the Movement method will run, if we would put return false we wouldn't be able to move

![image](https://user-images.githubusercontent.com/61652884/121044306-ba16da80-c7b5-11eb-9d31-0b19e1d8b32d.png)

Now we need to patch this

Go back inside the public class MainClass
inside the Awake method we will need to define the MovementPatch class for harmony like this
harmony.PatchAll(typeof(MovementPatch));

We will also log something with the logger we made like this
log.LogInfo("Tutorial Mod: Mod loaded");

this is how the MainClass.cs file should look like now

![image](https://user-images.githubusercontent.com/61652884/121046017-2f82ab00-c7b6-11eb-8c32-c8d959e68b9f.png)

now you can build the solution and copy the dll inside the plugins folder in the Bepinex folder but before we do that you might want to make it compatible with script engine to ease development and allow you to reload the mod without restarting the entire game

Add a new method inside the MainClass

private void OnDestroy()

inside this method we just unpatch harmony like this

harmony.UnpatchSelf();

also make sure you unload any resources etc https://github.com/BepInEx/BepInEx.Debug for more help with script engine

if you'd like to access something like the logger from a Prefix or Postfix method you do this like this

MainClass.instance.log

# On Your Own

https://docs.bepinex.dev/v5.4.4/articles/dev_guide/plugin_tutorial/index.html
https://github.com/BepInEx/HarmonyX/wiki
https://discord.com/invite/MpFEDAg

