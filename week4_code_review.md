# Code review

This document aims to describe the work done during week four, which consisted in attempting 10 code review challenges.

## Performing a Code Review

After performing 10 code reviews on some code snippets we had to choose the one that we thought was done better and comment further on it. This is the one I chose, it simulates a simple environment with people and departments:

```cs
class HumanResources
 {
    static void Main(string[] args)
     {
         Person person = new Person();
         person = person.GetManager();
     }
 }
 
 
 class Person
 {
     public Department Department { get; set; }
     public Person GetManager()
     {
         return Department.GetManager();
     }
 }
 
 
 class Department
 {
     private readonly Person _manager;
     public Department(Person manager)
     {
         _manager = manager;
     }
     public Person GetManager()
     {
         return _manager;
     }
 }
```

After analysing the code I came to a few conclusions:
- First of all it would always be nice to see the classes' access level, which, along with a few of the following points, does not really adhere with the **C# standard coding conventions**.
- The "GetManager()" method inside the "Person" class can be removed, that is because it could simply call the Department class' "GetManager()" method. That is a repetition but also extra not-needed work, therefore it does not satisfy the **KISS** and **DRY** principles.
- The first class has no reason to be called "HumanResources" since it does not do anything other than creating a Person object. This kind of violates the **YAGNI** principle, since it is not needed.
- Finally it goes against the **Open/Closed Principle (OCP)** because not every person is a manager, so if I wanted to create a Person instance that is a manager the code would break since a manager does not have a manager, I would have to change previously written code to asjust it.

## Responding to a Code Review

Here follows the code written by me:
```cs
class Program
 {
    static void Main(string[] args)
     {
         var department = new Department();
         var manager = new Manager();
         var employee = new Employee();
         department.SetManager(manager);
         employee.SetDepartment(department);
     }
 }
 
 abstract class Person
 {
     public Department Department { get; set; } = null;
     public void SetDepartment(Department department) {}
 }
 
 class Employee : Person
 {   
     public void SetDepartment(Department department)
     {
         if(Department == null)
         {
             Department = department;
         }
     }
 }
 
  class Manager : Person
 {
     public void SetDepartment(Department department)
     {
         if(Department == null)
         {
             Department = department;
             Department.SetManager(this);
         }
     }
 }
 
 class Department
 {
     public Person Manager;
     
     public Department()
     {
         Manager = null;
     }
     
     public void SetManager(Manager manager)
     {
         if(Manager == null)
         {
             Manager = manager;
             Manager.SetDepartment(this);
         }
     }
 }
```

This code addresses all the principle's violations and code smells that I spotted during my code review. Here follows what I did to make this code functioning and open for extension.<br>
- I renamed the "HumanResources" class to "Program" because it was not doing anything important or human resources' related. I also added public access level to each class.
- I created an abstract class called "Person" which only contains a "Department" property, and two more classes called "Employee" and "Manager" that inherit from it, that way I can create both without breaking anything. Also neither implements a "GetManager()" metohd.
- Finally the Department class' constructor sets the manager propery to an empty instance of a Manager, so it can be set later on. 

## Missed code smells

According to the results I misinterpreted a code smell: in fact I considered the "GetManager()" method in the "Person" class of the untouched code, to be a "Duplicated Code" code smell for the fact that a method with the same name already existed and could be easily accessed from the Person class itself. The right choice was **"Middle man"** because the "HumanResources" class was using the person's "GetManager()" method when it could have used the person's department's "GetManager()" method, without the need of a middle man. I feel like both are right answers, it is doing something via a middle man but it is also repeating code.