# 如何重写Object对象中的equals和hashcode

## 背景

对象比较判断两个对象相等，需要调用object.equals方法，如一个用户的对象

```java
User a = new User();
a.name = xiaoming;

User b = new User();
b.name = xiaoming;

a = b ?
```

a和b两个对象相等吗，看起来相等，但是实际上相等吗，实际上不是，a和b不相等。为了实现看起来相等，必须重写equals。

Java collection类中HashMap和HashSet等底层类，需要使用hashCode来做相等的判断。

基于这两点需求，自定义的类必须（保险起见）实现自定义的hashCode与equals方法。

## 条件约束

equals方法重写原则:

* 自反性（reflexive）。对于任意不为null的引用值x，x.equals(x)一定是true。

* 对称性（symmetric）。对于任意不为null的引用值x和y，当且仅当x.equals(y)是true时，y.equals(x)也是true。

* 传递性（transitive）。对于任意不为null的引用值x、y和z，如果x.equals(y)是true，同时y.equals(z)是true，那么x.equals(z)一定是true。

* 一致性（consistent）。对于任意不为null的引用值x和y，如果用于equals比较的对象信息没有被修改的话，多次调用时x.equals(y)要么一致地返回true要么一致地返回false。

* 对于任意不为null的引用值x，x.equals(null)返回false。

hashCode方法重写原则:

* 在一个Java应用的执行期间，如果一个对象提供给equals做比较的信息没有被修改的话，该对象多次调用hashCode()方法，该方法必须始终如一返回同一个integer。

* 如果两个对象根据equals(Object)方法是相等的，那么调用二者各自的hashCode()方法必须产生同一个integer结果。

* 并不要求根据equals(java.lang.Object)方法不相等的两个对象，调用二者各自的hashCode()方法必须产生不同的integer结果。然而，程序员应该意识到对于不同的对象产生不同的integer结果，有可能会提高hash table的性能。

## 正确实现Object的equals和hashCode

1. 17和31散列码

```java
public class User {
    private String name;
    private int age;
    private String passport;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof User)) {
            return false;
        }
        User user = (User) o;
        return user.name.equals(name) &&
                user.age == age &&
                user.passport.equals(passport);
    }
    //Idea from effective Java : Item 9
    @Override
    public int hashCode() {
        int result = 17;
        result = 31 * result + name.hashCode();
        result = 31 * result + age;
        result = 31 * result + passport.hashCode();
        return result;
    }
}
```

1. SDK 1.7以及后续版本实现

```java
public class User {
    private String name;
    private int age;
    private String passport;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof User)) {
            return false;
        }
        User user = (User) o;
        return Objects.equals(name, user.name) &&
                user.age == age &&
               Objects.equals(passport, user.passport);
    }
    @Override
    public int hashCode() {
        Object.hash(name, age, passport);
    }
}
```

1. Apache Commons Lang

```java
public class User {
    private String name;
    private int age;
    private String passport;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof User)) {
            return false;
        }
        User user = (User) o;
        return return new EqualsBuilder()
                .append(age, user.age)
                .append(name, user.name)
                .append(passport, user.passport)
                .isEquals();
    }
    @Override
    public int hashCode() {
        return new HashCodeBuilder(17, 37)
                .append(name)
                .append(age)
                .append(passport)
                .toHashCode()
    }
}
```

以上方法的原理都一样，sdk1.7版本以及之后的版本更加简单，Apache Commons Lang提供的方法比较优雅