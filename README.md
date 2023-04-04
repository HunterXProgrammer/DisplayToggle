# Description
Turn ON/OFF the display of your Android phone, like scrcpy, using ADB Shell or Root.

Check **[Reddit Tasker](https://www.reddit.com/r/tasker/comments/12bcdnj/project_share_turn_display_onoff_dont_disturb/)** for Tasker users.

Termux users can continue to the next section.

# Building
You can follow these steps to build it in **[Termux](https://f-droid.org/en/packages/com.termux/)**.

    pkg upgrade -y

&nbsp;

    pkg install -y wget openjdk-17 dx android-tools

&nbsp;

    cd; wget -O android.jar "https://github.com/Sable/android-platforms/blob/master/android-30/android.jar?raw=true"

&nbsp;

    nano DisplayToggle.java

Now copy paste this:-

```
import android.os.Build;
import android.os.IBinder;
import java.lang.reflect.Method;
import java.lang.reflect.InvocationTargetException;
public class DisplayToggle {

    private static final Class<?> CLASS;
	static {
        	try {
            		CLASS = Class.forName("android.view.SurfaceControl");
		} catch (ClassNotFoundException e) {
        	throw new AssertionError(e);
     	 }
    }

    public static void main(String... args) throws Exception {
	System.out.print("Display mode: "+args[0]);
	Method method = CLASS.getMethod("setDisplayPowerMode", IBinder.class, int.class);
	try {
            method.invoke(null, getBuiltInDisplay(),Integer.parseInt(args[0]));
        } catch (InvocationTargetException | IllegalAccessException e) {
	    e.printStackTrace();
        }
    }

    private static Method getGetBuiltInDisplayMethod() throws NoSuchMethodException {
       Method getBuiltInDisplayMethod;
            // the method signature has changed in Android Q
            // <https://github.com/Genymobile/scrcpy/issues/586>
            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.Q) {
                getBuiltInDisplayMethod = CLASS.getMethod("getBuiltInDisplay", int.class);
            } else {
                getBuiltInDisplayMethod = CLASS.getMethod("getInternalDisplayToken");
            }
        return getBuiltInDisplayMethod;
    }

    public static IBinder getBuiltInDisplay() {
        try {
            Method method = getGetBuiltInDisplayMethod();
            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.Q) {
                // call getBuiltInDisplay(0)
                return (IBinder) method.invoke(null, 0);
            }
            // call getInternalDisplayToken()
            return (IBinder) method.invoke(null);
           } catch (InvocationTargetException | IllegalAccessException | NoSuchMethodException e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

Then press **CTRL+X+Y+ENTER** and save it.

And then create its Java CLASS file:-

    javac -Xlint:none -source 1.7 -target 1.7 -cp android.jar DisplayToggle.java

Then DEX it using:-

    dx --dex --output DisplayToggle.dex DisplayToggle.class

You have compiled the DisplayToggle.dex file (around 2.5kb).

Now copy it to internal storage to use it later.

    termux-setup-storage

&nbsp;

    cp -f DisplayToggle.dex /storage/emulated/0

# How To Use It

In an ADB Shell (from Termux or PC) type:-

    adb shell cp -f /storage/emulated/0/DisplayToggle.dex /data/local/tmp

#### To Turn  Display OFF

    adb shell CLASSPATH=/data/local/tmp/DisplayToggle.dex app_process / DisplayToggle 0

#### To Turn  Display ON

    adb shell CLASSPATH=/data/local/tmp/DisplayToggle.dex app_process / DisplayToggle 2

#### Note:-

In Termux if you are rooted, you can just replace every `adb shell` with `su -c`.

# Credits

**[rom1v](https://blog.rom1v.com/2018/03/introducing-scrcpy/#run-a-java-main-on-android) - Method to make java code executable**

**[CheerfulPianissimo](https://github.com/Genymobile/scrcpy/issues/2888#issuecomment-1452140829) - Java code to make this possible**
