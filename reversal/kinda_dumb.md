# Challenge: Kinda Dumb

You're given 4 files: `kernel.cl`, `Main.java`, `Thing.java`, and `ThingThread.java`

A short look at the loop in Main reveals that it creates a bunch of ThingThreads, which each attempt to validate your string with the strings it generates.

In `ThingThread.java`, we see this comparison:

```java

if (s.equals(that.check)){

...

found = str;
break;
}
```

followed by the verification:

```java
if (found.equals("p32uq")){
    System.out.println("flag{" + that.check + "}");
} else {
    System.out.println("wrong");
}
```

where that.check is the string we write as input.

So, remove the comparison and move the verification into the for loop with where the comparison initially was, relocating the break statement as well (and remove that annoying 'wrong' statement). This means that we can continue to verify other strings until we're done.

Run the program with LWJGL's OpenCL loaded, and you will retrieve the flag shortly.