# JVM-Architecture
## The basic architecture <hr>
   ![JVM architecture-mainComponents](https://user-images.githubusercontent.com/36560845/107139410-1e7b7780-6945-11eb-99ff-25611653aee3.png)

## Full architecture <hr>
  ![full jvm architecture](https://user-images.githubusercontent.com/36560845/107154944-d178bf00-699f-11eb-94de-b2c334f36ba3.png)
  
## Garbaze Collection 
* If an object doesn't have any reference variable, it's garbez and eligible for GC. 
* **4 ways to make an object eligible for GC:**
  * **Nullifying the reference variable :**  ``` Service service = new Service() ; service = null ; ```  This service object is now eligible for GC. 
  * **Re-assigning the reference variable to another object** : 
  ![GC](https://user-images.githubusercontent.com/36560845/109393283-e915e880-794a-11eb-877a-367c78314b08.png)

  ```
  1. Service servcie1 = new Service() ; 
  2. Servcie service2 = new Service() ; 
  3. service1 = new Service() ;  
  4. service2 = service1 ; 
  ```
  * **Creating objects inside a method i.e. local variables:**
  ```
  class Test {
    public static void main(String[] args) {
      getService() ; 
      }
      public static void getService() {
      // service1 and service2 are local variables. so, both of them are eligible for GC.
        Service service1 = new Service() ;  
        Service service2 = new Service() ; 
       }
     }     
  ```
  * **Special Scenario**
  ```
  class Test {
    public static void main(String[] args) {
      Service service = getService() ; // we're capturing service1 from getService(). 
      }
      public static Service getService() {
      // service1 and service2 are local variables. so, both of them are eligible for GC.
        Service service1 = new Service() ;  
        Service service2 = new Service() ; ---------> eligible for GC
        return service1 ; 
       }
     }
  ```
  * **Island of Isolation:** 
  After line 9, though 3 objects have their internal references via i, we can't access them. These group of objects are considered in island. So, GC will clean up them all.
  ```
  class Test {
    Test i ; 
    public static void main(String[] args) {
       1. Test test1 = new Test() ; 
       2. Test test2 = new Test() ; 
       3. Test test3 = new Test() ; 
       
       4. test1.i = test2 ; 
       5. test2.i = test3 ; 
       6. test3.i  = test1 ; 
       7. test1 = null ; //------ test1 reference gone, but this object can still be accessible via test3.i ;
       8. test2 = null ; // ----- test2 reference gone, but this object can still be accessible via test3.i.i ; 
       9. test3 = null ; // ----- test3 reference gone. Now, we can't access any of them. So 3 objects are eligible for GC
      } 
    }
  ```
  ![IslandOfIsolation](https://user-images.githubusercontent.com/36560845/109395711-fdacad80-7957-11eb-91a4-8167d1eb3efd.png)

* We can only make objects eligible for GC. But, the cleaning task is done by JVM and it varies from JVM to JVM. We can't exactly specify when these objects will be cleaned up by JVM.
* **2 Methods to request JVM to run GC process**
    * Using System class: System class contains a static method gc() to clean up garbage. we can request JVM to perform garbez collection process using ```System.gc() ; ``` . We can't clean up garbage, we can only request JVM to clean up using this gc() method. ```System.gc()``` internally calls Runtime gc() method. 
    ```
    public class System { 
      public static void gc() {
         Runtime.getRuntime().gc() ; 
       }
     }
    ```
    * Using Runtime class: A java application can communicate with JVM using Runtime instance. Runtime is a singleton class. Runtime class contains an instance method gc() to clean up garbage objects.
    ```
    Runtime runtime = Runtime.getRuntime() ; 
    for(int i = 0 ; i < 10000 ; i++) {
      Date date = new Date() ; 
      date = null ;
      // at this point, 10000 objects are eligible for GC
     }
    runtime.gc() ; // request JVM to cleanup garbage
    ```
* Garbage Collector always calls ***finalize()*** method on that specific object before destroying that object to perform some tasks like resource deallocation, disconnect network or database connectivity etc. ***finalize()*** method is present in Object class. ``` protected void finalize() throws Throwable { // empty implementation } ```
* ***finalize()*** method doesn't destroy objects. Its called by GC only for doing cleanup activities.
* If a String is eligible for GC, finalize() of String class will be called. If a Sample class instance is eligible for GC, finalize() of Sample class will be called.
```
class Test {
  public static void main(String[] args) {
    String string = new String("shuvo") ;
    string = null; 
    Test test = new Test() ;
    test = null ; // now, finalize() of this Test class will be called.
    System.gc() ; 
   }
   public void finalize() {
    System.out.println("It will not be printed. Because, This finalize() method is an instance method of Test class. But, GC is calling finalize() mehtod of String class as string is eligible for GC") ; 
}
```
* We can call finalize() method explicitly. It'll be used as an instance method. No objects will be destroyed. Cleanup activities will be performed.
* If we call finalize() method ourselves, If exception occurs and there is no correspoiding catch block, JVM will terminate the program abnormally by throwing that exception. But, If Garbage collector calls finalize(), JVM will ignore uncaught exceptions.
* Even though an object is eligible for GC multiple times, GC will call finalize() method on that object only once. gc() can be called multiple times.
```
class FinalizeDemo {
  static FinalizeDemo finalizeDemo ; 
  public static void main(String[] args) {
    FinalizeDemo fd = new FinalizeDemo() ; 
    fd = null ; // fd is eligible for GC 
    System.gc() ; // calling finalize() and assigning
                  // this object reference to finalizeDemo 
                  // static object. 
    Thread.sleep(5000) ; 
    finalizeDemo = null ; // finalizeDemo is now eligible for GC
    System.gc() ;  // this time, finalize() won't be called twice. GC will destroy this object
    Thread.sleep(10000) ; 
   }
   public void finalize() {
    finalizeDemo = this ; 
   }
``` 
* **Memory leaks:** Objects which are created but not used anywhereand are not eligible for GC either.
* **Memory Management Tools to identify Memory Leaks:**
    - HP-PATROL
    - HP-J-Meter
    - HP-OVO
    - IBM-Tivoli
    - JPROBE
   
   
## Processes involved with Garbage collection
   - **Mark**: Start from root node of application, walks through the object graph and marks objects that are reachable.
   - **Delete/Sweep**: Delete unreachable objects
   - **Compact**: After removing the unreachable objects (by gc) holes. This step compacts the memory by moving the alive objects contiguously rather than keeping them fragmented.
   
## Generational Collectors
![jvm_heap_java8](https://user-images.githubusercontent.com/36560845/227317591-168004c9-74ba-4057-a8a9-32ae9dd259e6.png)

* Young Generation: Divided into following 2 spaces.
  - Eden: New spaces are created here
  - Survivor Space: When eden space is full, a minor gc is triggered and all the reachable objects are moved to survivor space. Remaining unreachable objects are then collected by garbage collector.
  
* Old Generation:
  - The objects that are survived even after multiple cycle of garbage collection, are moved to this space. 
