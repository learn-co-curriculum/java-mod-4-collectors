# Collectors

## Learning Goals

- Explain collectors in Java.
- Use collectors in stream pipelines.

## Introduction

The `collect` method along with the `stream.Collectors` utility provide
different ways of collecting, aggregating, and grouping values. We’ll look at
each of these in the following sections.

## Collecting Elements in a Collection

The `toList`, `toSet`, and `toMap` methods can be used to collect elements from
streams. We will be using the following `User` class in most of our examples.

```java
class User {
    private String name;
    private int age;
    private int distanceTravelled;
    private String state;

    public User(String name, int age, int distanceTravelled, String state) {
        this.name = name;
        this.age = age;
        this.distanceTravelled = distanceTravelled;
        this.state = state;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public int getDistanceTravelled() {
        return distanceTravelled;
    }

    public String getState() {
        return state;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", distanceTravelled=" + distanceTravelled +
                ", state=" + state +
                '}';
    }
}
```

Let’s look at a `toList` method example to understand how to collect elements.

```java
class Example {
    public static void main(String[] args) throws Exception {
        List<User> users = new ArrayList<>();
        users.add(new User("John" , 24, 2300, "NY"));
        users.add(new User("Rico" , 62, 2500, "TX"));
        users.add(new User("Jane" , 37, 5600, "VA"));
        users.add(new User("Kimi" , 45, 6700, "FL"));
        users.add(new User("Momo" , 58, 5400, "CA"));

        List<String> userNames = users.stream()
                                    .map(User::getName)
                                    .collect(Collectors.toList());

        System.out.println(userNames);
				// Output: [John, Rico, Jane, Kimi, Momo]
    }
}
```

Note that we could have used the `toList` method after `map` directly but we’re
using `collect` for demonstrating its usage.

Here’s how we would use `toList` directly:

```java
List<String> userNames = userList.stream()
                                    .map(User::getName)
                                    .toList();
```

The `Collectors` class also provides a `toCollection` method that allows us to
specify collections other than lists, sets, or maps.

```java
List<String> userNames = userList.stream()
                                    .map(User::getName)
                                    .collect(Collectors.toCollection(LinkedList::new));
```

## Aggregating Results into a Single Value

There are several methods that accumulates or aggregates stream elements into a
single value. Here are some of the methods you’re likely to use:

- `summingInt`, `summingLong`, `summingDouble`: These sum up stream elements
  after mapping them to an `Integer`/ `Long` / `Double` value.
- `minBy`, `maxBy`: Finds the minimum or maximum of all stream elements based on
  the provided `Comparator`.
- `averagingInt`, `averagingLong`, `averagingDouble`: These calculate the
  arithmetic mean of the stream elements based on the provided `ToIntFunction`.
  The return value is always a `double`.
- `counting`: This method counts the number of elements in the stream.

Let’s look a few examples:

```java
class Example {
    public static void main(String[] args) throws Exception {
        List<User> users = new ArrayList<>();
        users.add(new User("John" , 24, 2300, "NY"));
        users.add(new User("Rico" , 62, 2500, "TX"));
        users.add(new User("Jane" , 37, 5600, "VA"));
        users.add(new User("Kimi" , 45, 6700, "FL"));
        users.add(new User("Momo" , 58, 5400, "CA"));

        int totalDistanceTravelledByUsers = users.stream()
                                            .collect(Collectors.summingInt(user -> user.getDistanceTravelled()));
        System.out.println(totalDistanceTravelledByUsers); // 22500

        double averageUserAge = users.stream()
                                    .collect(Collectors.averagingInt(user -> user.getAge()));
        System.out.println(averageUserAge); // 45.2

        Optional<User> youngestUser = users.stream()
                                        .collect(Collectors.minBy(comparing(User::getAge)));
        System.out.println(youngestUser.get()); // User{name='John', age=24, distanceTravelled=23000, state=NY}

        long usersTravelledOver5000 = users.stream()
                                        .filter(user -> user.getDistanceTravelled() > 5000)
                                        .collect(Collectors.counting());
        System.out.println(usersTravelledOver5000); // 3
    }
}
```

## Dividing Elements into Groups

The `partitionBy` and `groupingBy` methods divides stream elements into two or
more groups. Let’s look at examples of both of these methods.

### Partitioning

The `partitionBy` method divides elements into a `true` and `false` group based
on the `Predicate` passed to the method. The return type is
`Map<Boolean, List<T>>`.

```java
class Example {
    public static void main(String[] args) throws Exception {
        List<User> users = new ArrayList<>();
        users.add(new User("John" , 24, 2300, "NY"));
        users.add(new User("Rico" , 62, 2500, "TX"));
        users.add(new User("Jane" , 37, 5600, "VA"));
        users.add(new User("Kimi" , 45, 6700, "FL"));
        users.add(new User("Momo" , 58, 5400, "CA"));
        users.add(new User("Nana" , 38, 5400, "FL"));
        users.add(new User("Aria" , 29, 4400, "FL"));
        users.add(new User("Moor" , 42, 2400, "CA"));

        Map<Boolean, List<User>> partitionedUsers = users.stream()
                                                    .collect(Collectors.partitioningBy(
                                                            user -> user.getDistanceTravelled() > 4000));
        System.out.println(partitionedUsers);
    }
}
```

```java
{
    false=[User{name='John', age=24, distanceTravelled=2300, state=NY}, User{name='Rico', age=62, distanceTravelled=2500, state=TX}, User{name='Moor', age=42, distanceTravelled=2400, state=CA}],
    true=[User{name='Jane', age=37, distanceTravelled=5600, state=VA}, User{name='Kimi', age=45, distanceTravelled=6700, state=FL}, User{name='Momo', age=58, distanceTravelled=5400, state=CA}, User{name='Nana', age=38, distanceTravelled=5400, state=FL}, User{name='Aria', age=29, distanceTravelled=4400, state=FL}]
}
```

The `true` key contains a `List` of `User` objects that satisfy the condition
passed to the `partitioningBy` method. The `false` key contains a `List` of
`User` objects which did not match the condition.

### GroupingBy

This method divides the elements into multiple groups based on a classification
function that maps elements to keys. Here’s the function signature of the
method:

```java
groupingBy(Function<? super T, ? extends K> classifier)
```

Let’s group our users by their state:

```java
class Example {
    public static void main(String[] args) throws Exception {
        List<User> users = new ArrayList<>();
        users.add(new User("John" , 24, 2300, "NY"));
        users.add(new User("Rico" , 62, 2500, "TX"));
        users.add(new User("Jane" , 37, 5600, "VA"));
        users.add(new User("Kimi" , 45, 6700, "FL"));
        users.add(new User("Momo" , 58, 5400, "CA"));
        users.add(new User("Nana" , 38, 5400, "FL"));
        users.add(new User("Aria" , 29, 4400, "FL"));
        users.add(new User("Moor" , 42, 2400, "CA"));

        Map<String, List<User>> partitionedUsers = users.stream()
                                                    .collect(Collectors.groupingBy(User::getState));
        System.out.println(partitionedUsers);
    }
}
```

```java
{
	TX=[User{name='Rico', age=62, distanceTravelled=2500, state=TX}],
	FL=[User{name='Kimi', age=45, distanceTravelled=6700, state=FL}, User{name='Nana', age=38, distanceTravelled=5400, state=FL}, User{name='Aria', age=29, distanceTravelled=4400, state=FL}],
	VA=[User{name='Jane', age=37, distanceTravelled=5600, state=VA}],
	NY=[User{name='John', age=24, distanceTravelled=2300, state=NY}],
	CA=[User{name='Momo', age=58, distanceTravelled=5400, state=CA}, User{name='Moor', age=42, distanceTravelled=2400, state=CA}]
}
```

### Downstream Collector

Both the `partitioninBy` and `groupingBy` methods have another parameter called
`downstream`. This is a collector that is applied to the map values of a
predicate or a classifier function. Let’s look at a couple of examples.

For the first example, we will take our `partitioningBy` example from above and
get the total distance travelled as the map value instead of the list of users.

```java
class Example {
    public static void main(String[] args) throws Exception {
        List<User> users = new ArrayList<>();
        users.add(new User("John" , 24, 2300, "NY"));
        users.add(new User("Rico" , 62, 2500, "TX"));
        users.add(new User("Jane" , 37, 5600, "VA"));
        users.add(new User("Kimi" , 45, 6700, "FL"));
        users.add(new User("Momo" , 58, 5400, "CA"));
        users.add(new User("Nana" , 38, 5400, "FL"));
        users.add(new User("Aria" , 29, 4400, "FL"));
        users.add(new User("Moor" , 42, 2400, "CA"));

        Map<Boolean, Integer> partitionedUsers = users.stream()
                .collect(Collectors.partitioningBy(
                        user -> user.getDistanceTravelled() > 4000,
                        Collectors.summingInt(User::getDistanceTravelled)
                ));
        System.out.println(partitionedUsers);
    }
}
```

```java
{
	false=7200,
	true=27500
}
```

Here’s what the output looked like in our example without the downstream
collector:

```java
{
    false=[User{name='John', age=24, distanceTravelled=2300, state=NY}, User{name='Rico', age=62, distanceTravelled=2500, state=TX}, User{name='Moor', age=42, distanceTravelled=2400, state=CA}],
    true=[User{name='Jane', age=37, distanceTravelled=5600, state=VA}, User{name='Kimi', age=45, distanceTravelled=6700, state=FL}, User{name='Momo', age=58, distanceTravelled=5400, state=CA}, User{name='Nana', age=38, distanceTravelled=5400, state=FL}, User{name='Aria', age=29, distanceTravelled=4400, state=FL}]
}
```

After adding the downstream collector, the lists are reduced to a single value
using the `summingInt` method.

Similarly, the `groupingBy` method can also use a downstream collector. We add a
downstream collector to our previous `groupingBy` example so that the user list
only contains user names instead of the full user object strings.

```java
class Example {
    public static void main(String[] args) throws Exception {
        List<User> users = new ArrayList<>();
        users.add(new User("John" , 24, 2300, "NY"));
        users.add(new User("Rico" , 62, 2500, "TX"));
        users.add(new User("Jane" , 37, 5600, "VA"));
        users.add(new User("Kimi" , 45, 6700, "FL"));
        users.add(new User("Momo" , 58, 5400, "CA"));
        users.add(new User("Nana" , 38, 5400, "FL"));
        users.add(new User("Aria" , 29, 4400, "FL"));
        users.add(new User("Moor" , 42, 2400, "CA"));

        Map<String, List<String>> partitionedUsers = users.stream()
                .collect(Collectors.groupingBy(
                        User::getState,
                        Collectors.mapping(User::getName, Collectors.toList())
                ));
        System.out.println(partitionedUsers);
    }
}
```

```java
{
	TX=[Rico],
	FL=[Kimi, Nana, Aria],
	VA=[Jane],
	NY=[John],
	CA=[Momo, Moor]
}
```

We are using the `mapping` collector to map the list of `User` objects into a
list of user names (`String`). This is a great way to shape the data for easier
representation and manipulation.

## Conclusion

We have learned about several collector methods and also learned how to divide
elements based on predicates or classification functions.
