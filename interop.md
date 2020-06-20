# Sharing Clojure code with Java developers

It is probably a matter of taste, but the
Here are the five easy steps to deliver a function written in Clojure so that
Java programmers can call it as if were a function written in Java and included
in a .jar file:

1. Create a Java-like project with [Leiningen](https://leiningen.org/)
1. Write a (very thin) wrapper function for your function
1. Use the `ns` form at the top of your source file to export your function
1. Edit your `project.clj` file to use AOT compilation
1. Build an uberjar file

## Our example

To illustrate these steps we assume that you work at the mythical Example
company and promise your Java colleagues to provide a function called `plus` in
a package called `com.example.cl4j`.

## 1 - Create the project

Use Leiningen to create a project designed to play nicely with Java:

```
lein new com.example.cl4j
```
Note that Leiningen created your project in a directory called
`com.example.cl4j` and your "main" source file is a little deeper in the
directory tree than usually.

## 2 - Write a wrapper for your function

Open your "main" source file (`com.example.cl4j/src/com/example/cl4j.clj`) an
write your `plus` function and a second one called `-plus` which acts as kind of
a wrapper.

```
(ns com.example.cl4j)

(defn plus
  "calculate a + b"
      [a b]
          (+ a b))

(defn -plus
  "a Java-callable wrapper around the `plus` function"
    [a b]
      (plus a b))
```

### 3 - Export your function
Now edit the `(ns ...)` form at the top of the file to generate `.class` files
and export your function:

```
(ns com.example.cl4j
  (:gen-class
   :name com.example.cl4j
   :methods  [#^{:static true} [plus [int int] int]]))

(defn plus
  "calculate a + b"
    [a b]
    (+ a b))

(defn -plus
  "a Java-callable wrapper around the `plus` function"
  [a b]
  (plus a b))

```

Your source file is now ready to go.

### 4 - Configure the project
The configuration is just adding a line (`:aot :all`) to your `project.clj` file:

```
(defproject com.example.cl4j "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "EPL-2.0 OR GPL-2.0-or-later WITH Classpath-exception-2.0"
            :url "https://www.eclipse.org/legal/epl-2.0/"}
  :dependencies [[org.clojure/clojure "1.10.1"]]
  :aot :all
  :repl-options {:init-ns com.example.cl4j})
```

### 5 - Build an uberjar file

Use Leiningen to create a jar file which contains everything your Java
colleagues will need:

```
lein uberjar
```

## That's it!
You're done! Java programmers now just have to add the jar file (called
something like
`com.example.cl4j/target/com.example.cl4j-0.1.0-SNAPSHOT-standalone.jar`) when
building and running their program.

Calling your `plus` function in a Java program is pretty conventional:

```java
package com.example.something;

import com.example.cl4j;

public class Client {
    public static void main(String [] args) {
        System.out.println(cl4j.plus(4, 4));
    }
}
```

> **Credits**: this page is based on [the answer provided by StackOverflow user
> clartaq](https://stackoverflow.com/questions/2181774/calling-clojure-from-java)
