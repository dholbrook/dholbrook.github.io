---
layout: post
title: "Scala Password Hash"
date: 2014-01-03 13:39:19 -0800
comments: true
categories: scala security

---

I'm working on a spray framework REST based tech demo project, and I want to securely store passwords as salted hashes in a database.  I took a look around at existing projects, and couldn't find quite what I was looking for.

- simple to use and understand
- based on JVM built in crypto libraries
- written in Scala

The closest thing I found was this: [Java PBKDF2 Password Hashing Code](https://crackstation.net/hashing-security.htm#javasourcecode) 

But it is written in Java.... So I took a crack at porting it to Scala.

It lives here on github:

https://github.com/dholbrook/scala-password-hash

The public methods `createHash()` and `validatePassword()` are for the most part the same as the Java version.  Some of the private methods are quite a bit different.

### slowEquals ###

This is an interesting method in the Java implementation.  It is designed to always take the same amount of time to determine equality regardless of the length of the arrays being compared. Additionally it uses the XOR operator `^` to produce consistent branching.  I suspect using XOR in Java this way is somewhat controversial, regardless I used a similar approach in my Scala implementation.

``` java Java: slowEquals()
    private static boolean slowEquals(byte[] a, byte[] b)
    {
        int diff = a.length ^ b.length;
        for(int i = 0; i < a.length && i < b.length; i++)
            diff |= a[i] ^ b[i];
        return diff == 0;
    }
```
The most significant change in the Scala port is making `diff` immutable.  The same result is achieved by folding over a range, and using an accumulator seeded by the initial test of `a.length ^ b.length`.

``` scala Scala: slowEquals()
    private def slowEquals(a: Array[Byte], b: Array[Byte]): Boolean = {
      val range = 0 until scala.math.min(a.length, b.length)
      val diff = range.foldLeft(a.length ^ b.length) {
        case (acc, i) => acc | a(i) ^ b(i)
      }
      diff == 0
    }
```

### fromHex / toHex  ###

The utility functions `fromHex` and `toHex` saw a big size reduction.

``` java Java: fromHex() and toHex()

    private static byte[] fromHex(String hex)
    {
        byte[] binary = new byte[hex.length() / 2];
        for(int i = 0; i < binary.length; i++)
        {
            binary[i] = (byte)Integer.parseInt(hex.substring(2*i, 2*i+2), 16);
        }
        return binary;
    }

    private static String toHex(byte[] array)
    {
        BigInteger bi = new BigInteger(1, array);
        String hex = bi.toString(16);
        int paddingLength = (array.length * 2) - hex.length();
        if(paddingLength > 0) 
            return String.format("%0" + paddingLength + "d", 0) + hex;
        else
            return hex;
    }

```

``` scala Scala: fromHex() and toHex()

    private def fromHex(hex: String): Array[Byte] = 
      hex.sliding(2, 2).toArray.map(Integer.parseInt(_, 16).toByte)

    private def toHex(bytes: Array[Byte]): String = 
      bytes.map("%02X" format _).mkString

```

### Testing ###

I tested the code by using it in conjunction with the existing Java implementation.  Verifying that passwords created by each side were interoperable with the other.  I used [ScalaCheck](http://www.scalacheck.org/) to randomly generate a set of passwords to test.

```scala PasswordHashTest
    package scalapasswordhash
    
    import org.scalacheck.Gen
    import org.scalacheck.Gen._
    import org.scalatest.FunSuite
    import org.scalatest.prop.GeneratorDrivenPropertyChecks
    import passwordhash.{PasswordHash => JPasswordHash}
    
    class PasswordHashTest extends FunSuite with GeneratorDrivenPropertyChecks  {
    
      test("hash generated from Java should validate with Scala") {
        forAll (identifier) { (password: String) =>
          val h1 = JPasswordHash.createHash(password)
          assert(PasswordHash.validatePassword(password, h1))
        }
      }
      
      test("hash generated from Scala should validate with Java") {
        forAll (identifier) { (password: String) =>
          val h1 = PasswordHash.createHash(password)
          assert(JPasswordHash.validatePassword(password, h1))
        }
      }  
        
    }
```