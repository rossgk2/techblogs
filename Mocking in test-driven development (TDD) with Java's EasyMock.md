In this article, we'll explore the test-driven development practice of *mocking*.

Consider a class `Cls` with a method `method()` that relies upon a method `helperMethod()`, where `helperMethod()` queries some external resource, and suppose that our *goal* is to test whether `method()` works as intended.

```java
public class Cls
{
    private Object helperMethod(Object args)
    {
        // Use some external resource to obtain "result".
        return result;
    }
    
    public Object method(Object args) 
    { 
               // Calculate "result" by using helperMethod(args) somehow.
               return result; 
    }
}
```

Since `method()` calls `helperMethod()`, a method that relies on an unpredictable external resource, we will need to imitate, or *mock*, `helperMethod()` in order to achieve our goal. Instead of actually calling `helperMethod()` within `method()`, we will make an educated guess as to what `helperMethod()`'s output should be for various inputs.

To prepare for imitating `helperMethod()` in this way, we will replace the call to `helperMethod()` with a call to an interface method.

```java
public interface HelperI { Object helperMethod(Object args); }

public class Cls
{
    private HelperI helperI;
    public void setHelperI(HelperI helperI) { this.helperI = helperI; }
    
    public Object method(Object args)
    {
        // Calcluate "result" by using helperI.helperMethod(args) somehow.
        return result;
    }
}
```

Specifically, the above code replaces the call to `helperMethod()` with a call to `helperI.helperMethod()`.

Now, we can use a library such as `EasyMock` to provide a good "best guess" implementation of the interface `HelperI` and, most importantly, its method `helperMethod()`.

Here's an implementation that uses `EasyMock` and `JUnit` to do exactly this.

```java
/* We omit the necessary "static import" statements for the EasyMock and JUnit libraries to reduce clutter. */

public class Tester
{
    private HelperI helperI;
    private Cls cls;
    @Before
    /* @Before is a JUnit annotation, not an EasyMock annotation. Any method tagged with @Before is executed 
    before each test case. */
    public void setUp() throws Exception
    {
        /* For an interface intf, createNiceMock(intf) returns an instance of 
        a class that implements intf, where all abstract methods of intf are 
        implemented by using default values for return values.
        Note, createNiceMock() does come from EasyMock. */
        helperI = createNiceMock(HelperI.class);
        cls = new Cls();
        cls.setHelperI(helperI);
    }

    @Test
    /* @Test is a JUnit annotation that indicates the method to which it is attatched is to be executed as a 
    test case. */
    public void testMethod() // Recall, our goal is to test whether method() works.
    {
        /* Now, "block out" helperMethod(). The expect() call below specifies that, 
        for i in {1, ..., n}, the ith time helperI.helperMethod() is called, it should 
        have recieved the input args[i] (in this example, the input will be coming from method(),
        since method() calls helperI.helperMethod()), and that it will return returns[i]. */

        n = ... // some positive integer
        Object[] args = ... // A length n array of inputs. We will use EasyMock to 
                           // ensure that helperI.helperMethod() recieves args[i] 
                          // from method() in the ith JUnit test.
            
        Object[] helperReturns = ... // A length n array of outputs. We will use EasyMock to impose that
                                    // helperI.helperMethod() should return helperReturns[i] upon recieving args[i] as input.
            
        Object[] expectedMethodReturns = ... // A length n array of outputs. We hope that method() will 
                                            // return expectedMethodReturns[i] in the ith iteration.
            
        for (i = 0; i < n; i ++)
            expect(helperI.helperMethod(args[i])).andReturn(helperReturns[i]);
        
        /* Apply the mocking that was just specified above to the helperI interface. */
        replay(helperI);
        cls.setHelperI(helperI);

        /* The mocking is all set up now, so we can now test if method() works. */
    
        /* Do the n tests that were set up by the expect() calls above. 
        The ith iteration of the for loop executes the ith test. 
        In the ith test, the input passed to helperI.helperMethod() from method()
        should be args[i]. When the input to helperI.helperMethod() is indeed 
        args[i], the output helperI.helperMethod() will return returns[i]. */
        
       	for (i = 0; i < n; i ++)
            Object args = ... // args should be such that, in the ith iteration of this loop, calling
                             // cls.method(args) results in calling helperI.helperMethod(args[i]) within cls.method()
            assertEquals(expectedMethodReturns[i], cls.method(args)); // assertEquals() is a JUnit method
    }
}
```

##### Source

[This tutorial](https://www.vogella.com/tutorials/EasyMock/article.html) was used as a source for this article.
