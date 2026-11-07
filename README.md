# WARNING: this is WIP, nothing here has been made with readability in mind 

Below are my current notes

### Overall principle:

Lets take a cryptographic approach.

"No lock is unpickable" is true in the sense that there exists a physical shape from a known set of keys that will open it.
But this is also true for cryptographic primitives: any cryptographic scheme (be it sym/asym, encryption/signautre, etc) that relies on private keys.  yet we have satisfying definition of security for those.

Lets push the analogy further. a lock can be seen as asymmetric authentication primitive, ie, a MAC.
In a MAC, there is a verification algorithm that knows the key, and which must accept an input only if it is made with the same key. 

In simplified terms, A MAC scheme is parameterized by a set of keys, and is made of two algorithms:
 - SIGN(message, key) which takes a key and genereates an authentication toke,
 - VERIFY(mesage, token, key) which takes a token, a key, and returns "accepted" or "rejected"
The security of the MAC requires that for any key, it is not possible to find a message and a token that are accepted by VERIFY without knowning the key.

in that analogy, the key has the same role as a lock key, and the lock is the VERIFY algorithm with the key already dialled in.

Like I've said, it's always possible to know if a key is the right one by testing trivailly possible to break security by trying out all keys. In cryptograhy, a "computationally secure" device is vulnerable to this attack. In fact, the definition of computational security is that any attack takes (roughly) as much time as trying all the keys. The rationale being, it is easy to make the number of possible keys unfathomably large, making this attack inpractical.

Everything in the above statement can be applied to locks instead of cryptography. Thus, my argument is "computational security" is a suitable security definition for locks too. Of course, like in cryptography, some assumptions may be required to be able to prove that some system satsify this security definition.

I believe that no current lock satisfy this definition (even with generous assumptions) because they all leak some (linear amount of) information about the private key when probed. I propose a few families of designs that satisfy it, + some additional security and practicality considerations.






On an abstract level: 
We have the equivalent of the "pin", ie, the thing which has as many possible states (in this case, its position) as there are keys, and each manufactured pin has a specific state that, when reached, allows the lock to open. In normal usage, the pin state is changed by the key, which mean that the attacker is also free to manipulate its state. The crux of a good design is to prove the lock secure *with minimal assumptions on how the state can be manipulated*. In particular, the speed at which each state can be tried directly implies a upper bound on attack time.

The key is an element from the key space, whose only job is  to manipultate the pin to the correct position.

the driver pin is the thing that is actionned when attempting to open the lock. It is going to interact with the pin, and result in a position that blocks opening if the pin is not a the correct state.


### Security.

Here the assumption that drives the design of this lock:
Let two separate physical parts X, Y, and let x,y be two possible positions of X and Y respectively.  Any physical implementation of a "position(X)=x  AND position(Y)=y" is imperfect, ie, it will also leak "position(X)=x OR position(Y)=y".

(note: the equality  may be replaced by ~= )
By "leak", I mean that it is possible to distinguish this state from the others. 

This statement can be proved in practice for a given lock, the assumption here is that it is always possible to do so.

Using this hypothesis, it is easy to prove that any lock for which the "pin" is made of multiple parts connected with "AND"s cannot be secure. The proof consists in exhibiting a typical lock-picking attack where each part of the pin is moved independently and the hypothesis is used to determinine whether one part is at the right position. Iterating this let us find to correct position of each part independently from the others, and hence the attack time is only the sum of the number of part positions instead of the product. In a sense, the assumption of imperfect "AND" is just the justification that lock-picking is always possible for a very large class of locks (in this case, all locks that I'm aware of).

Note that the attack is practical only if the tests of the pin's position can be made quickly enough.

For instance, applied to StuffMadeHere's lock, the imperfection in the "AND" is in the fact that forcefully operating the lock should result in a sligthly bigger rotation when one of the pin is aligned with its gate. In this case the attack is made longer by the fact that each try has to include the time to test the pin *and* lift them, rotate them to the gate, and bring them back to the accessible chamber.

Thus, one way of being immune to the lock-picking attack is to have only a single pin. The downside is that it still needs to have enough possible states (ie, keys) so that brute-force attacks are unpractical.


### mechanical design.


possible basic 2d shapes for the pin/driver: 
 -ball (2 rotation dims)
 -cylinder (1 rotation, 1 translation)
 -rectangle (2  translation)


The constraint is that we want to manipulate this surface using a key to an (arbitrary) position, with precision, repeatability, wear-resistance. The surface is proportional to the key space size and should be maximised. 


translation is inconvenient because to move to a known location you need at least twice the size of the dimension. rotation doesn't have this issue, but I don't know how to manipulate a ball with a key as required. -> cylinder


either the pin has a hole and the driver pin is a needle, or the opposite. I don't see how the latter could be workable.

the key must tightly couple to another piece "plug" to have the correct position relative to the pin

the driver pin is going to be forced by an attacker to be as reliable as possible on hole detection

there is no way around the fact that if the pin is not in the correct state, then only it will prevent the driver from moving. We can have a huge force reduction to ensure that it cannot be broken.
one way is a track on the housing driving the driver position, with appropriate slope for the reduction
to compensate forcing, the track could have a spring loaded trapdoor to a dead end that open when too much force is applied. complicated.
alternatively, the housing is a cam and the driver a follower, with a spring. risky?
alternatively, the driver could be a screw that would turn with the housing as a gear. complicated.
OR, make it so that breaking the pin leads to a locked and un-unlockable lock.
Current approach : make the driver pin strong enough (eg 3mm diameter). Its insertion into the hole is done with a spring with no external access so it cannot be used to damage the key pin

### manufacturing constraints:

size of key space is D^2 where D is the sole parameter := min distance between pin and hole that guarantees pin does not enter hole. 
D ~= max allowance + extra to prevent chipping off pin. independent from pin diameter -> make pin diameter as large as possible.
primary objective is tigh tolerances, because the key space is essentially  pin size/tolerance

therefore the key, pin and plug should all be one solid part (and not a assembly). as a result I'll try to rely only on cylinders on a lathe and drilled.

I'm not caring a lot about non-covert attack, but as a rule of thumb any destructive attack should requiring drilling though at least 2mm of steel.

key_pin spring :   https://www.leespring.com/compression-springs?search=CI010EG13M ... too thin?
driver_pin spring: https://www.leespring.com/compression-springs?search=CI008C02M


----------------------------
##   other design

instead of relying of a single pin, we can keep multiple pins but instead test them sequentially one after the other. Algorithmically, this is like having a temporary variable x, and computing x <- (x AND pin) while iterating on pin. The physical element that tests the pins positions can only test one pin at a time, and has an internal state that is set iff the current pin isn't correct. That element is moved from pin to pin, and when it reaches the end, the lock will only open if the internal state is not set. When repositioning the testing element, the internal state is reset.
Lets call A the following assumption: when testing a pin, it must not be possible to detect whether the internal state has been set or not. Assuming A, there is no way to recover information about the key from manipulation of pins, because the testing element erases the information about the already tested pins (it only stores the AND of the already tested pins).
Thus the design is focused on making A a trustworthy assumption.


Note: one can generalise this idea by observing that the lock can be made to compute anything (mechanically, or not even). For instance, one could embed the verification algorithm of a digital signature scheme. In the case where the computation is done electronically, this is equivalent to digital locks. Though it would be interesting to avoid the issue of digital locks (EM-based interference) by using an alternative basis for computation, eg microfluidics, see below


----------------------------
##   other (Dank) design


 the computation can be purely analogic. with hydro power the key could simply a pressure value, ie, a piston is moved by a given amount by the key, which moves the piston at the other end of the tube, lock unlocks only when at a specific position.
To avoid testing every position very quickly, the testing procedure can close the pressure tube before testing of the value, and could even "reset" the input side of the tube to force re-entering the key




----------------------------
## lower bound on the constants: Adding delay

everything above focuses on making a lock that is asymptotically secure (ie attack_time = O(key_space_size) ), but doesn't address the constants in the equation. for crypto this is a smaller issue because each step is so short, but in physical reality the number of steps isn't so big so the constants may be more important.
In our case, all lock attacks essentially use two operations: set pins, test pins (the latter potentially leaks info). Thus there are only two time constants, one for each operation.

==> introduce an unskipable delay on either operation. This is similar to the idea of Verifiable Delay Functions in crypto (or, more rudimentarily, increased rounds on pwd hashing).
The test pins operation seems more suitable.

