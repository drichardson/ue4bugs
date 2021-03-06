## Status

Last Tested: 4.26. Bug still occurs. Based on note below, unlikely to be fixed.

Discovered in 4.21

Tracked by [UE-67922](https://issues.unrealengine.com/issue/UE-67922).

According to this note in DelegateCombinations.h (from git commit
555cddd7e4232870c7aebc2417e1793919b0420c), DECLARE_EVENT appears to be
deprecated:

```c++
/**
 * Declares a multicast delegate that is meant to only be activated from OwningType
 * NOTE: This behavior is not enforced and this type should be considered deprecated for new delegates, use normal multicast instead
 */
#define DECLARE_EVENT( OwningType, EventName ) FUNC_DECLARE_EVENT( OwningType, EventName, void )
```


## Report

As of UE4 b70f31f6645, DECLARE_EVENT produces a class with a public Broadcast()
call, which means it can be called from anywhere. The [documentation on
Events](https://docs.unrealengine.com/en-us/Programming/UnrealArchitecture/Delegates/Events)
says:

> Events are very similar to multi-cast delegates . However, while any class can bind events, only the
> class that declares the event may invoke the event's  Broadcast, IsBound, and Clear functions. This
> means event objects can be exposed in a public interface without worrying about giving external classes
> access to these sensitive functions. Event use cases include including callbacks in purely abstract 
> classes, and restricting external classes from invoking the  Broadcast, IsBound, and Clear functions.

I took a look at the change logs and I believe this is a regression introduced
in git commit e710837e2abc286c921dac03411d479ef414a2ac. In that commit, the
Broadcast() method is moved from a protected access to public access. This
renders the DECLARE_EVENT macro (which makes the owner class a friend)
pointless.

Here is the commit log:

```
commit e710837e2abc286c921dac03411d479ef414a2ac
Author: Steve Robb <Steve.Robb@epicgames.com>
Date:   Fri Aug 28 11:51:02 2015 -0400

    Removed Broadcast forwarding function for derived multicast delegates to bring it into parity with the variadic version.

    #codereview robert.manuszewski,michael.troughton

    [CL 2672383 by Steve Robb in Main branch]
```
