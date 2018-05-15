# Composite key handling, using @Idclass annotation in Spring boot java

This blog is totally implementation based. If you are confuse how to visualize
composite key relationship in Spring data JPA, this blog is for you.

As far as we use relational databases we have to consider key fields which act
as identifiers of the table. Other than usual primary keys, composite key plays
specific role since multiple candidates collectively generate Id.

In JPA (Java Persistence API) there's two ways to specify entity composite
keys:`@IdClass` and `@EmbeddedId`

Among those this blog basically explain how we handle composite key relationship
in Java spring boot application using @Idclass annotation. First let's begin
after cloning my git hub repository.
[https://github.com/bhagyaj/composite-idclass](https://github.com/bhagyaj/composite-idclass)

This will basically based on below one to many relationship between Employee and
Department.

![](https://cdn-images-1.medium.com/max/1600/1*cbhwPB6b__kElasmzvYcug.png)
<span class="figcaption_hack">Department-Employee</span>

Now I'm gonna explain how this relationship with composite keys implement in
Spring boot application written in Java. First we have to create Employee and
Department model classes separately. And two separate classes for Composite
keys. To be clear Employee entity will be used as the example.

```java
@Entity
@Table(name = "Employee")
@IdClass(EmployeeId.class)
public class Employee {
    @Id
    @Column(name = "Name")
    private String name;
    @Id
    @Column(name = "Department")
    private String departmentName;
    @Column(name = "Designation")
    private String designation;
    @Id
    @Column(name = "DepartmentLocation")
    private String departmentLocation;
    @ManyToOne
    @JoinColumns({
            @JoinColumn(name="Department", referencedColumnName="Name",insertable = false,updatable = false),
            @JoinColumn(name="DepartmentLocation", referencedColumnName="Location",insertable = false,updatable = false),
    })
    @JsonIgnore
    private Department department;
    @Column(name = "Salary")
    private Double salary;
```

In above code extract @Idclass annotation is used to introduce composite Id
class (EmployeeId) to the entity class. @Id is used to show variables which are
part of composite key.

@ManyToOne shows department entity has multiple employees and the fields that
used to join those(Fields available in both entities). Insertable and updatable
false because it's not a responsibility of an employee to create and update
Department.

Let's see how id class looks like with composite key. As below it only has key
fields without any annotation.

```java
public class EmployeeId implements Serializable {
        private String name;
        private String departmentName;
        private String departmentLocation;

}
```

When we use @Idclass annotation it is really easy to query data without using
the name of the composite Id class. For example if we want to retrieve employee
name we can use below query.

    select e.name from Employee e

It's directly accessing Id fields without accessing idclass. Meantime we can use
this method to apply for legacy systems since we don't have to change existing
model class.

Let's see how mapping look likes in the department end(One department for
multiple employees).

```java
@Entity
@Table(name = "Department")
@IdClass(DepartmentId.class)
public class Department {
    @Id
    @Column(name = "Name")
    private String name;
    @Id
    @Column(name = "Location")
    private String location;
    @Column(name = "EmpNumber")
    private Integer empNumber;
    @OneToMany(fetch = FetchType.
, cascade = {CascadeType.
}, mappedBy = "department")
    private Set<Employee> employees;
```

@OneToMany says one department has multiple employees. fetch type depict whether
we load everything at once(EAGER) or we load on the fly (LAZY). mappedBy
introduce the object of the Department class available in mapped class Employee.
Employees belong to that department will fetch to the declared set.

I'll conclude my blog by presenting final output when I request department by
its composite Id and Employee By its composite Id.

**Request: Department by Id**

```json
{
 "name": "RnD",
 "location": "Kalutara",
 "empNumber": 5,
 "employees": [
 {
 "name": "Bhagya",
 "departmentName": "RnD",
 "designation": "Architect",
 "departmentLocation": "Kalutara",
 "salary": 500000
 },

{
 "name": "Singam",
 "departmentName": "RnD",
 "designation": "Engineer",
 "departmentLocation": "Kalutara",
 "salary": 50000
 }
 ]
}
```

**Request: Employee by Id**

```json
{
 "name": "Bhagya",
 "departmentName": "RnD",
 "designation": "Architect",
 "departmentLocation": "Kalutara",
 "salary": 500000
}
```

Stay tune for next blog post to see how `@EmbeddedId used to represent composite
key.`

* [Java](https://medium.com/tag/java?source=post)
* [Spring Boot](https://medium.com/tag/spring-boot?source=post)
* [Spring Data](https://medium.com/tag/spring-data?source=post)
* [Jpa](https://medium.com/tag/jpa?source=post)
* [Microservices](https://medium.com/tag/microservices?source=post)

From a quick cheer to a standing ovation, clap to show how much you enjoyed this
story.

### [Bhagya Rupasinghe](https://medium.com/@bhagyajayashani)
