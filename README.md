# AndroidInterviewAdvance

<h3>What is Coroutine?</h3> -
coroutines help to manage long-running tasks that might otherwise block the main thread and cause your app to become unresponsive. 
Over 50% of professional developers who use coroutines have reported seeing increased productivity.
This topic describes how you can use Kotlin coroutines to address these problems,
enabling you to write cleaner and more concise app code.

<h1>What is difference between async and await ?</h1>
Both launch and async are the functions in Kotlin to start the Coroutines.

launch{}
async{}
The difference is that the launch{} returns a Job and does not carry any resulting value whereas the async{} returns an instance of Deferred<T>, which has an await() function that returns the result of the coroutine like we have future in Java in which we do future.get() to get the result.

In other words:

launch: fire and forget.
async: perform a task and return a result.
Let's take an example to learn launch vs async.

We can use the launch as below:

```
val job = GlobalScope.launch(Dispatchers.Default) {
    // do something and do not return result
}
```

It returns a job object which we can use to get a job's status or to cancel it.

In the above example of launch, we have to do something and NOT return the result back.

But when we need the result back, we need to use the async.

```
val deferredJob = GlobalScope.async(Dispatchers.Default) {
    // do something and return result, for example 10 as a result
    return@async 10
}
val result = deferredJob.await() // result = 10
```

Here, we get the result using the await().

In async also, we can use the Deferred job object to get a job's status or to cancel it.

Note: I have used GlobalScope for quick examples, we should avoid using it at all costs. In an Android project, we should use custom scopes based on our usecase such as lifecycleScope, viewModelScope and etc.

Another difference between launch and async is in terms of exception handling.

If any exception comes inside the launch block, it crashes the application if we have not handled it.

However, if any exception comes inside the async block, it is stored inside the resulting Deferred and is not delivered anywhere else, it will get silently dropped unless we handle it.

Let's understand this difference with code examples.

Suppose we have a function that does something and throws an exception:

```
private fun doSomethingAndThrowException() {
    throw Exception("Some Exception")
}
```
Now using it with the launch:

```
GlobalScope.launch {
    doSomethingAndThrowException()
}
```
It will CRASH the application as expected.

We can handle it as below:

```
GlobalScope.launch {
    try {
        doSomethingAndThrowException()
    } catch (e: Exception) {
        // handle exception
    }
}
```
Now, the exception will come inside the catch block and we can handle it.

And now, using it with the async:

```
GlobalScope.async {
    doSomethingAndThrowException()
}
```
The application will NOT crash. The exception will get dropped silently.

Again, we can handle it as below:

```
GlobalScope.async {
    try {
        doSomethingAndThrowException()
    } catch (e: Exception) {
        // handle exception
    }
}
```
Now, the exception will come inside the catch block and we can handle it.

Let me tabulate the difference between launch and async.

<b>Launch</b>
Fire and forget. <br>
launch{} returns a Job and does not carry any resulting value.	<br>
If any exception comes inside the launch block, it crashes the application if we have not handled it.	<br>

<b>Async</b>
Perform a task and return a result.<br>
async{} returns an instance of Deferred<T>, which has an await() function that returns the result of the coroutine.<br>
If any exception comes inside the async block, it is stored inside the resulting Deferred and is not delivered anywhere else, it will get silently dropped unless we handle it.<br>


<h3>• How can two distinct Android apps interact? </h3>

Intents: 
Apps can communicate with each other using intents. An app can send an intent to request an action from another app, such as opening a specific activity or sharing data. The receiving app can then handle the intent and respond accordingly.

Content Providers: 
Content providers allow apps to share data with each other. An app can define a content provider to make its data accessible to other apps. Other apps can then query or modify this data using content resolver methods.

Broadcasts: 
Apps can send broadcast messages to notify other apps of events or trigger actions. Apps can register to receive specific broadcasts using intent filters.

Bound Services: 
Apps can bind to a service provided by another app to interact with it. This allows for more complex and ongoing interactions between apps.

Shared Preferences: 
Apps can use shared preferences to store and retrieve small amounts of data in key-value pairs. If two apps use the same shared preferences file, they can share data through this mechanism.

File Sharing: 
Apps can share files with each other using file providers or by writing to a shared directory that both apps have access to.

It's important to note that some of these methods require permissions and may have security implications. Apps should use these methods responsibly and follow best practices for inter-app communication.

<h3>• Is it possible to run an Android app in multiple processes? </h3>

Yes, it is possible to run an Android app in multiple processes. This can be achieved by setting the android:process attribute in the <application> tag of the app's manifest file. By doing this, you can specify a different process name for components of your app, such as activities, services, or broadcast receivers, allowing them to run in separate processes.

However, it's important to note that running an app in multiple processes can have implications for performance and resource usage. Each additional process consumes memory and CPU resources, so it's generally recommended to use multiple processes only when necessary and to carefully consider the trade-offs involved.

</h3>• What is AIDL? Enumerate the steps in creating a bounded service through AIDL.? </h3> <br>

AIDL (Android Interface Definition Language) is a language specifically developed by Google for defining the interface requirements between a client and server in Android's inter-process communication (IPC). It allows you to define the programming interface that both the client and server agree upon in order to communicate with each other.

To create a bounded service through AIDL, you need to follow these steps:

Define the AIDL interface: Create an .aidl file that defines the methods that the client can call on the service. This file should be placed in the src/main/aidl directory of your Android project. For example, if your service is named MyService, the AIDL file could be named IMyService.aidl.

aidl
```
package com.example;

interface IMyService {
    int add(int a, int b);
}
```

Implement the AIDL interface in the service: Create a service that implements the AIDL interface. This service should extend android.app.Service and provide an implementation for each method defined in the AIDL interface.

```
public class MyService extends Service {
    private final IMyService.Stub binder = new IMyService.Stub() {
        @Override
        public int add(int a, int b) {
            return a + b;
        }
    };

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
}
```

Declare the service in the manifest: Add the service to your AndroidManifest.xml file, specifying the AIDL interface that the service implements.

```
<service android:name=".MyService">
    <intent-filter>
        <action android:name="com.example.IMyService"/>
    </intent-filter>
</service>
```

Bind to the service from the client: In the client app, bind to the service using an Intent that specifies the AIDL interface.


```
public class MainActivity extends AppCompatActivity {
    private IMyService myService;

    private final ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            myService = IMyService.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            myService = null;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Intent intent = new Intent();
        intent.setComponent(new ComponentName("com.example", "com.example.MyService"));
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(connection);
    }
}
```
Call methods on the service: Once the client is connected to the service, you can call methods on the service using the interface defined in the AIDL file.

```
if (myService != null) {
    try {
        int result = myService.add(5, 3);
        Log.d(TAG, "Result: " + result);
    } catch (RemoteException e) {
        e.printStackTrace();
    }
}
```
These steps outline the basic process of creating a bounded service in Android using AIDL for inter-process communication.


<h3>• What is a ContentProvider and what is it typically used for? </h3>

A ContentProvider is a component in Android that manages access to a central repository of data. It provides a standard interface for interacting with the underlying data source, which can be a SQLite database, a file system, an in-memory data structure, or any other persistent storage mechanism. ContentProviders are typically used for sharing data between different applications, though they can also be used within a single application to manage access to shared data.

<b>ContentProviders offer several key features:</b>

<b>Data Abstraction:</b> <br>
ContentProviders abstract the underlying data source, allowing clients to interact with the data without needing to know its specific implementation details.

<b>Data Security:</b> <br>
ContentProviders can enforce permissions to control access to the data, ensuring that only authorized clients can read or write data.

<b>Data Sharing:</b> <br>
ContentProviders enable data sharing between different applications. By using a ContentProvider, an application can make its data accessible to other applications, subject to the permissions it defines.

<b>Data Querying:</b> <br>
ContentProviders support querying data using a structured query language (SQL)-like syntax, making it easy to retrieve specific subsets of data.

<b>Data Modification:</b> <br>
ContentProviders support data modification operations, such as inserting, updating, and deleting data, ensuring that changes to the data are managed consistently.

Overall, ContentProviders play a crucial role in the Android platform's architecture by providing a standardized way to manage and share data between different parts of an application or between different applications.

<h1> How to find duplicates in array in kotlin ?</h1>
We can use any function from the following to remove duplicates from an array in Kotlin:

```
distinct()
toSet()
toMutableSet()
toHashSet()
```

Let's start learning one by one by example.

Consider a data class Mentor like below:

data class Mentor(val id: Int, val name: String)
And, an array of Mentor:

```
val mentors = arrayOf(
    Mentor(1, "Amit Shekhar"),
    Mentor(2, "Anand Gaurav"),
    Mentor(1, "Amit Shekhar"),
    Mentor(3, "Lionel Messi"))
```

Remove duplicates using distinct()
In Kotlin, we can use distinct() function available in Collection functions to remove duplicates.

```
val distinct = mentors.distinct()
println(distinct)
```

This will print the following:

```
[Mentor(id=1, name=Amit Shekhar),
Mentor(id=2, name=Anand Gaurav),
Mentor(id=3, name=Lionel Messi)]
```

Note:

Maintain the original order of items.
Among equal elements of the given array, only the first one will be present in the output.
Returns List
Here, as we have used it to remove duplicate mentors from the array, similarly, we can use it to remove duplicate strings from an array.

Remove duplicates using toSet()
In Kotlin, we can use toSet() function available in Collection functions to remove duplicates.

```
val toSet = mentors.toSet()
println(toSet)
```

This will print the following:

```
[Mentor(id=1, name=Amit Shekhar),
Mentor(id=2, name=Anand Gaurav),
Mentor(id=3, name=Lionel Messi)]
```

Note:

Maintain the original order of items.
Returns Set which is a read-only set. It means that we can't perform operations like add on the set. Next, we will see toMutableSet() which returns read/write set.
Remove duplicates using toMutableSet()
In Kotlin, we can use toMutableSet() function available in Collection functions to remove duplicates.

```
val toMutableSet = mentors.toMutableSet()
println(toMutableSet)
```

This will print the following:
```
[Mentor(id=1, name=Amit Shekhar),
Mentor(id=2, name=Anand Gaurav),
Mentor(id=3, name=Lionel Messi)]
```

Note:

Maintain the original order of items.
Returns MutableSet which is a read/write set. It means that we can perform operations like add on the mutable set.
Remove duplicates using toHashSet()
In Kotlin, we can use toHashSet() function available in Collection functions to remove duplicates.

```
val toHashSet = mentors.toHashSet()
println(toHashSet)
```

This will print the following:

```
[Mentor(id=3, name=Lionel Messi),
Mentor(id=1, name=Amit Shekhar),
Mentor(id=2, name=Anand Gaurav)]
```

Note:

Similar to MutableSet but do NOT maintain the original order of items.
Returns HashSet
Here, as we have used it to remove duplicate mentors from the array, similarly, we can use it to remove any duplicate elements like strings, numbers, etc from an array.


<h1>what is open keyword in kotlin?</h1>

In Kotlin, we can mark a class, a function, or a variable with the open keyword like below:

Class
```
open class Mentor {

}
```

Function
```
open fun guide() {

}
```

Variable
```
open val slotsAvailable = 5
```

Let's understand why we use the open keyword in Kotlin.

open keyword with class
The open keyword with the class means the class is open for the extension meaning that we can create a subclass of that open class.

In Kotlin, all the classes are final by default meaning that they can not be inherited.

Suppose we have a class Mentor:

```
class Mentor {

}
```

We will not be able to create a subclass of the class.
```
class ExperiencedMentor: Mentor() {

}
```

The compiler will show an error.

This type is final, so it cannot be inherited from
If we want to allow inheritance, we need to mark the class with the open keyword. Basically, opening it for the extension.

Our updated Mentor class:

```
open class Mentor {

}
```

Now, we will be able to create a subclass of this class.

```
class ExperiencedMentor: Mentor() {

}
```

It will work perfectly.

You must have noticed that it is completely opposite to what we do in Java.

Let's see how it is different in Kotlin and Java.

```
In Java		In Kotlin
final class Mentor { }	->	class Mentor { }
class Mentor { }	->	open class Mentor { }
```

Basically, in Kotlin, all the classes are final by default but in Java, this is exactly the opposite. In Java, to make the class final, we need to use the final keyword.

That is why we need the open keyword in Kotlin with the class to allow inheritance.

open keyword with function
Similar to the classes, all the functions in Kotlin are by default final meaning that you can't override a function.

Consider a function guide() inside the Mentor class:

```
open class Mentor {

    fun guide() {

    }

}
```

We will not be able to override the function.

```
class ExperiencedMentor : Mentor() {

    override fun guide() {

    }

}
```

The compiler will show an error.

guide' in 'Mentor' is final and cannot be overridden
In order to allow the overriding of the function, we need to add the open keyword.

Our updated Mentor class:

```
open class Mentor {

    open fun guide() {

    }
}
```

We will be able to override the function now like below:

```
class ExperiencedMentor : Mentor() {

    override fun guide() {

    }
}
```

It will work perfectly.

Again, in Kotlin, all the functions are final by default but in Java, this is exactly the opposite. In Java, to make the function final, we need to use the final keyword.

That is why we need the open keyword in Kotlin with the function to allow it to be overridden.

open keyword with variable
Similarly, the variables in Kotlin are final by default. So, to override it in the child class, we need to set the variables as open in our base class.

Here is the base class Mentor:
```
open class Mentor {

    val slotsAvailable = 5

}
```

Here is the child class.
```
class ExperiencedMentor : Mentor() {

    override val slotsAvailable = 10

}
```

We will not be able to override the variable.

The compiler will show an error.

'slotsAvailable' in 'Mentor' is final and cannot be overridden
In order to allow the overriding of the variable, we need to add the open keyword.

Our updated Mentor class:
```
open class Mentor {

    open val slotsAvailable = 5

}
```

We will be able to override the variable now like below:

```
class ExperiencedMentor : Mentor() {

    override val slotsAvailable = 10

}
```

It will work perfectly.

Again, in Kotlin, all the variables are final by default but in Java, this is exactly the opposite. In Java, to make the variable final, we need to use the final keyword.

That is why we need the open keyword in Kotlin with the variable to allow it to be overridden.


<h1>What is internal modifier in kotlin ?</h1>
The declarations marked with the internal modifier are visible everywhere in the same module.

A module is a set of Kotlin files compiled together:

an IntelliJ IDEA module;
a Maven project;
a Gradle source set (with the exception that the test source set can access the internal declarations of main);
a set of files compiled with one invocation of the <kotlinc> Ant task.


// Internal.kt file

// Visible to everyone in the same module
```
internal const val numberThree = 3

// Visible to everyone in the same module
internal open class User() {
    // Visible to everyone in the same module that has visibility on User
    internal val numberEight = numberThree.plus(5)
}
```

// SameModule.kt file
```
// numberThree is visible because this file is in the same module than Internal.kt
// numberEight is visible because this file is in the same module than User
private const val numberEleven = numberThree.plus(User().numberEight)
```
// DifferentModule.kt file
```
// ERROR: numberThree is not visible because this file is in a different module than Internal.kt
// ERROR: numberEight is not visible because User() is in a different module than Internal.kt
private const val numberEleven = numberThree.plus(User().numberEight)
```

<h1>JvmStatic, JvmOverloads, JvmField Annotations in Kotlin</h1>
https://medium.com/@anandgaur22/jvmstatic-jvmoverloads-jvmfield-annotations-in-kotlin-d856fe38fd92


<h1> what is difference between letinit and lazy?</h1>
<b>lateinit</b>
<b>lazy</b>
Let's begin with the lateinit property.

<b>lateinit</b>
lateinit in Kotlin is useful in a scenario when we do not want to initialize a variable at the time of the declaration and want to initialize it at some later point in time, but we make sure that we initialize it before use.

One way(that is not a good way) to achieve this is by creating a nullable variable as below:

```
private var mentor: Mentor? = null
```

And as we all know that there is always a better way in Kotlin to achieve what we need.

What if we do not want to make the variable nullable?

The answer is lateinit.

Let's update our code:

```
private lateinit var mentor: Mentor
```

Here, we can notice that the lateinit keyword and also the variable is non-nullable.

That is what we wanted.

And later we can initialize the variable when we need it as below:

```
fun bookASlot() {
    mentor = Mentor()
}
```

But we must take care that we initialize the variable before accessing it, or else it will throw the following exception.

kotlin.UninitializedPropertyAccessException: lateinit property mentor has not been initialized
In Kotlin, we also have a way to check if the lateinit variable is initialized or not.

By using isInitialized:

```
if(this::mentor.isInitialized) {
    // access mentor
} else {
    // do something else
}
```

Things to consider when we use the lateinit property:

Can be only used with the var keyword.
Can be only used with a non-nullable variable.
Should be used if the variable is mutable and can be initialized later.
Should be used if you are sure about the initialization before use.
This was about the lateinit property in Kotlin.

Now, it's time to learn about the lazy property in Kotlin.

lazy
lazy in Kotlin is useful in a scenario when we want to create an object inside a class, but that object creation is expensive and that might lead to a delay in the creation of the object that is dependent on that expensive object.

```
class Session {
    private val mentor: Mentor = Mentor()
}
```

Suppose Mentor is an expensive object. And Session is the object that is dependent on the Mentor object.

If Mentor object creation takes time, it will delay the creation of Session object.

So, this is where the lazy keyword in Kotlin will help us.

Let's update our code:

```
class Session {
    private val mentor: Mentor by lazy { Mentor() }
}
```

Here, we have used the lazy keyword.

So, we need to understand that the object mentor will get initialized only when it is accessed for the first time, else it will not get initialized.

It will lead to the fast creation of the Session object in the above example because the Mentor object will not get initialized unnecessarily during the object creation of Session. It will get initialized when it is accessed for the first time.

Things to consider when we use the lazy property:

Can be only used with the val keyword, hence read-only property.
We want the variable to be initialized only if we need it for the first time.
Must understand that it only creates the object when we access it for the very first time and then in the subsequent access, it returns the same object.
This was about the lazy property in Kotlin.

Now we must have understood the lateinit vs lazy properties in Kotlin

