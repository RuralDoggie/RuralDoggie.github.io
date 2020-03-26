---
layout: article
title: 记一次 spring boot + JUnit Jupiter 单元测试开发经历
---

## 参数化测试
参数化测试是指多次调用 @Test 方法，但每次都使用不同的参数值。参数化测试必须使用 @ParameterizedTest 进行注解，而且必须为其参数指定一个来源。

JUnit Jupiter 提供了多个来源。每个来源指定一个 @ArgumentsSource，也就是一个 ArgumentsProvider 实现。本节将介绍如何使用 3 个来源：

@ValueSource
@EnumSource
@MethodSource

### @ValueSource
在 @ValueSource 中，您指定单个文字值数组，系统将这些文字值 — 一次一个地 — 提供给您的 @ParameterizedTest 方法。

语法类似于：
@ParameterizedTest
@ValueSource(longs = { 1L, 2L, 3L, 4L, 5L })
public void findById(Long id) {
  assertNotNull(classUnderTest);
  Person personFound = classUnderTest.findById(id);
  assertNotNull(personFound);
  assertEquals(id, personFound.getId());
}
首先您告诉 JUnit，findById() 方法是一个 @ParameterizedTest，如上面第 1 行所示。然后使用数组初始化器语法来指定数组，如第 2 行所示。JUnit 将调用 findById() 测试方法，每次将数组中的下一个 long 传递给该方法（第 3 行），直到用完数组。您可像任何 Java 方法参数一样使用该参数（第 5 行）。

作为数组名所提供的 @ValueSource 属性名必须全部采用小写，而且必须与其末尾有字母 s 的类型相匹配。例如，ints 与 int 数组匹配，strings 与 String 数组匹配，等等。

并不支持所有的原语类型，仅支持以下类型：

String
int
long
double

### @EnumSource
在 @EnumSource 中，您指定一个 enum，JUnit — 一次一个地 — 将其中的值提供给 @ParameterizedTest 方法。

语法类似于：
@ParameterizedTest
@EnumSource(PersonTestEnum.class)
public void findById(PersonTestEnum testPerson) {
  assertNotNull(classUnderTest);
  Person person = testPerson.getPerson();
  Person personFound = classUnderTest.findById(person.getId());
  assertNotNull(personFound);
  performPersonAssertions(person.getLastName(), person.getFirstName(), person.getAge(), person.getEyeColor(),
      person.getGender(), personFound);
}
首先您告诉 JUnit，findById() 方法是一个 @ParameterizedTest，如第 1 行所示。然后指定该 enum 的 Java 类，如第 2 行所示。JUnit 将调用 findById() 测试方法，每次将下一个 enum 值传递给该方法（第 3 行），直到用完该 enum。您可像任何 Java 方法参数一样使用该参数（第 5 行）。

注意，PersonTestEnum 类包含在本教程的配套示例应用程序中。它位于 com.makotojava.learn.junit 包中的 src/test/java 树中。

### @MethodSource
使用注解 @MethodSource，可以指定您喜欢的任何复杂对象作为测试方法的参数类型。语法类似于：
@ParameterizedTest
@MethodSource(value = "personProvider")
public void findById(Person paramPerson) {
  assertNotNull(classUnderTest);
  long id = paramPerson.getId();
  Person personFound = classUnderTest.findById(id);
  assertNotNull(personFound);
  performPersonAssertions(paramPerson.getLastName(), paramPerson.getFirstName(),
      paramPerson.getAge(),
      paramPerson.getEyeColor(), paramPerson.getGender(), personFound);
}
@MethodSource 的 names 属性用于指定一个或多个方法名，这些方法为测试方法提供参数。一个方法来源的返回类型必须是 Stream、Iterator、Iterable 或数组。此外，提供者方法必须声明为 static，所以不能将它用在 @Nested 测试类内（至少截至 JUnit 5 Milestone 5 时不能这么做）。

在上面的示例中，personProvider 方法（来自示例应用程序）类似于：
static Iterator<Person> personProvider() {
    PersonTestEnum[] testPeople = PersonTestEnum.values();
    Person[] people = new Person[testPeople.length];
    for (int aa = 0; aa < testPeople.length; aa++) {
      people[aa] = testPeople[aa].getPerson();
    }
    return Arrays.asList(people).iterator();
}
假设您想为测试方法添加一个额外的参数提供者。可以这样声明它：
@ParameterizedTest
@MethodSource(value = { "personProvider", "additionalPersonProvider" })
public void findById(Person paramPerson) {
  assertNotNull(classUnderTest);
  long id = paramPerson.getId();
  Person personFound = classUnderTest.findById(id);
  assertNotNull(personFound);
  performPersonAssertions(paramPerson.getLastName(), paramPerson.getFirstName(),
      paramPerson.getAge(),
      paramPerson.getEyeColor(), paramPerson.getGender(), personFound);
}
我们使用数组初始化器语法指定这些方法（第 2 行），而且将按您指定的顺序调用各个方法，最后调用的是 additionalPersonProvider()。

### @CsvFileSource
    @CsvFileSource(resources = "/test/case/order/order_create_success.csv", delimiter = '|', numLinesToSkip = 1)
![AltText](/asset/demo/demo.jpg)