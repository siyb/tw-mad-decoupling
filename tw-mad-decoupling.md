% Decoupled Application Development
% Patrick Sturm
% 13.10.2015

# Motivation

## Motivation 1
* Coupling is bad
	* Coupling means dependency
* Reusability and maintainability of software components is increased when coupling is decreased
* There are even metrics to rate code based on coupling
	* e.g. C(oupling) B(etween) O(bject) classes
	* [http://www.spinellis.gr/codequality/](http://www.spinellis.gr/codequality/) - good read!
* As we said before, Android applications are not monolithic
* We can create separate components
* We can fragment our UI and logic
* Why should we introduce dependencies?

## Motivation 2

* The concepts in these slides are very Fragment centric
* You can also apply most of the concepts to Activities and Services, but as mentioned before, you should really cut back on Activity use and utilize Fragments instead
* The techniques in these slides are opinionated - there are other way to achieve similar results
	* If you are planning to maintain a large codebase of different Apps with similar features
	* ... or if you are dealing with similar data throughout your app
	* these techniques might help you to reuse logic without relying on concrete feature implementations!
* Not a silver bullet that can be applied equally to all problems

## Motivation 3

* Large code base (for Android projects), close to 400.000 lines of code
* Several apps shared similar / the same features (~6)
	* Modular design (that was before we had gradle)
	* Modules have dependencies to certain functionality
	* Modules can be compiled arbitrarily
	* Different versions for Tablet / Phone

# Contracts

## Contracts - Introduction 1
* There are two different kinds of contracts (... that I want to show to you)
* Contracts that handle communication between components (i.e Fragments or Activities) ...
	* Contracts define protocols for component interaction
	* Contracts may offer methods to keep / check the contract more reliably
* ... and Contracts that define capabilities of components
	* Define a capability protocol
	* Help components to access the capabilities
	* Think of those contracts as mixins (Java 7 doesn’t support mixins, Java 8 -> default interface methods)

## Contracts - Introduction 2

* For CommunicationContracts, we need someone that is capable of receiving the communication
	* This can be another component (like a Fragment)
	* ... or something that will choreograph the application’s behavior
* ComponentContracts do not require a receiver, they just provide functionality that is shared by different components

## Contracts - CommunicationContracts 1

* Application specific coupling is moved to receivers
	* We are separating the concerns properly, our components are not aware of the navigation stack and how exactly they are combined or used
	* Application specific receivers handle choreographing
		* All the ugly boilerplate glue code is situated within those application specific classes
		* Changing stuff happens in one place (also SPOF – single point of failure – not always good)
		* Allows reusing components and increases maintainability
* Not only can we choreograph self made components, we can apply the same principle to native Android concepts
	* e.g. ActionBar / Toolbar - Have a receiver show / hide tabbed navigation on demand

## Contracts - CommunicationContracts 2

* On Android, broadcasts and BroadcastReceivers can be used to implement the contract / receiver “pattern”
* Since the protocols are meant to be used within the application, there is no need to fire global broadcasts, we can use LocalBroadcastManager to handle everything.
	* We have covered LocalBroadcastManager before!
* We need to make sure, that everybody that needs to listen to the communication listens to the communication, we also need to make sure that a component does not listen to our communication any more when it becomes inactive!
onResume() / onPause()

## Contract - CommunicationContracts - 3
```java
public class MyContract {
	public static final ACTION_SOMEACTION = 
		"package.class.ACTION_SOMEACTION"; 
	public static final INTENT_EXTRA_SOMEEXTRA =
		"package.class.INTENT_EXTRA_SOMEEXTRA"; 

	public static final void 
		sendIntent(Context c, Intent i) {
		LocalBroadcastManager.getInstance(c) 
		.sendBroadCast(i); 
	} 
	public static final boolean 
		isSomeActionIntent(Intent i) { 
		return i.getAction().equals(ACTION_SOMEACTION); 
	}
```

## Contract - CommunicationContracts - 4

```java
	public static final Intent getSomeIntent(int i) { 
		Intent i = new Intent(ACTION_SOMEACTION); 
		i.putExtra(INTENT_EXTRA_SOMEEXTRA, i); 
		return i; 
	} 
	public static final int 
		getSomeInfoFromIntent(Intent i) { 
		return i.getIntExtra(
			i.putExtra(INTENT_EXTRA_SOMEEXTRA); 
	} 
	// register / unregister methods here!
}
```

## Contract - ComponentContract - 1

```java
public class MyContract { 
    private static final String 
	BUNDLE_EXTRA_SOMETHING = ...; 
    private Fragment f; 
    public MyContract(Fragment f) { 
        this.f = f; 
    } 
    public setSomeThing(int i) { 
        f.getArguments()
		.putInt(BUNDLE_EXTRA_SOMETHING, i) 
    } 
    public getSomeThing() { 
    	return 	f.getArguments()
		.getInt(BUNDLE_EXTRA_SOMETHING);
    } 
}
```

## Contract - ComponentContract - 2

```java
public class MyFragment { 
    private MyContract myContract; 
    public static final MyFragment 
	newInstance(int someThing) { 
        MyFragment f = new MyFragment(); 
        Bundle args = new Bundle(); 
        f.setArguments(args); 
        f.myContract.setSomeThing(i); 
        return args; 
    } 
    // we need this constructor,
	// otherwise or app might crash
    public MyFragment() { 
        myContract = new MyContract(this); 
    }
}
```

## Contract - ComponentContract - 3

* I have chosen the Fragment example for a good reason: nested child fragments
* Using a contract in this manner allows us to implement the data storing logic once ...
* ... without having to deal with inheritance
	* Inheritance is "EVIL", unless we have mixins (e.g. Scala Traits)
	* Lets rephrase that: Composition should be used wherever possible
* Fragments use setArguments(...) to retain data (even if they are paused, etc), using setArguments is the preferred way to “persist” Fragment data temporarily

## Contract - ComponentContract - 4

* The factory method “newInstance” is an accepted pattern of the Android Fragment world, use it ;)
* Make sure to setArguments(…) before the Fragments enter their lifecycle, or you will have a bad time
Once a Bundle has been set using setArguments(…) it is safe to change its content

## Contract - CommunicationContracts - Contract - 1

```java
public class NavigationContract { 
	public static final String ACTION_NAVIGATION =; 
	public static final String INTENT_EXTRA_FRAGMENT =; 
	public enum FRAGMENT { 
		MYFRAGMENT 
	}
	public static final 
		getNavigationIntent(FRAGMENT f) { 
		Intent i = new Intent(ACTION_NAVIGATION); 
		i.putSerializable(INTENT_ACTION_FRAGMENT, f); 
		return i;
	}
```
## Contract - CommunicationContracts - Contract - 2

```java
	public static final void 
		sendNavigationIntent(Intent i,
			FRAGMENT f, Bundle extras) { 
		if (i.getExtras() != null) 
			i.putExtras(extras);
		LocalBroadcastManager.getInstance(c)
			.sendBroadcast(i); 
	} 
	public Bundle getMyFragmentExtras(int i) { 
		Bundle b = new Bundle(); 
		b.putInt(SOMECONST, i); 
		return b; 
	}

	// helper methods get fragment from intent, 
	// isNavigationIntent, etc
}
```

## Contract - CommunicationContracts - Receiver - 1

```java
public class NavigationReceiver { 
	private FragmentManager f; 
	public NavigationReceiver (FragmentManager f) { 
		this.f = f; 
	} 
	// can handle requests from anybody, even instances,
	// that have no accessto the fragment manager -> 
	// cleaner code, just one class needs a     
	// FragmentManager instance
	public void onReceive(Context c, Intent i) { 
		if (NavigationContract.isNavigationIntent(i)) { 
			FRAGMENT f = NavigationContract
			.getFragmentFromIntent(i);
```

## Contract - CommunicationContracts - Receiver - 2

```java
	Fragment f = FragmentFactory
		.getFragment(f, i.getExtras());
	// we can do much more here: backstack, 
	// animation, up button action, etc
	f.beginTransaction()
		.replace(R.id.container, f)
		.commit();
	} else { throw new BrokenContractException(); } 
}
```

## Contract - CommunicationContracts - Factory - 1

```java

// not a classical factory + we could also use an abstract factory!
// all the ugly contruction boilerplate goes here
public class FragmentFactory { 
	public static final Fragment
		getFragment(FRAGMENT f, Bundle extras) { 
		Fragment fragment; 
		switch(f) { 
			case MYFRAGMENT: 
			fragment = MyFragment
				.newInstance(
					extras.getIntExtra(SOMECONST); 
			break; 
		} 
		return fragment;
	} 
}

```

## Contract - CommunicationsContract - Implementation Detail - 1

* When using this pattern, Fragments should be unaware of each other, unless they have a parent/child Fragment relationship (the parent can know of the children, but not the other way around -> use ComponentContracts to share information)
* Each Fragment is environment agnostic and must not rely on other parts of the program to provide functionality or remove functionality (CommunicationContracts excluded)
* We can use onCreateOptionsMenu(...) or onResume() to create dependencies that are intrinsic to the Fragment itself (i.e. buttons in the ActionBar or navigation behavior)


## Contract - CommunicationsContract - Implementation Detail - 2

```java
public onCreateOptionsMenu(Menu menu,
	MenuInflater inflater) {
	// get rid of all the dirt left by others
	menu.clear(); 
	inflater.inflate(R.menu.myMenu, menu); 
	TabContract.disableTabNavigation(); 
	HamburgerContract.blockHamburgerMenu(); 
}
```
# Conclusion

## Conclusion - 1
* Enabling / Disabling (show/hide) tabbed navigation
* Enabling / Disabling Up button, up button Action
	* Also: Hamburger Menu vs. Up
* Spawning Dialogs
* ActionBar manipulation, i.e. hide / show
* Data update propagation
* Login / Logout events
* etc

## Conclusion - 2

* IMPORTANT: the only classes that know about ActionBar, Hamburger Menu, Navigation Tabs, Dialogs, etc, are the ones that receive the command
* If you want to reuse a component in an app where one component is not required (e.g. hamburger menu functionality), the receiver can just ignore all broadcasts OR even better, no receiver is registered to act on those broadcasts
* Event driven! (with all its pros and cons)

## Conclusion - 3

* NOT A SILVER BULLET!
* Pros:
	* "defensive" approach, since we are not depending on instances via interfaces / abstract classes, we are less likly to run into null pointers
	* receivers take the role of choreographers, rather than activities: less boilerplate in activities, more specialized (SRP) choreographers
	* broadcasts are received on an explicit basis, if a broadcast gets not listened to, there are very few implications
	* using predefined mediator facilities
	* more flexibility: add / remove functionality without changes classes (comp. interfaces)
* Cons:
	* boilerplate code
	* not exactly the typical Java approach, since we are using Intents to convey our actions
	* complexity is an issue and debugging can be tedious
	


