# pallet.stevedore

Stevedore is a DSL for generating shell scripts using clojure.

## Why?

While there is nothing wrong with normal shell scripting, Stevedore allows the
ability to easily interpolate clojure values. As a consequence, we gain the
benefits of being able to flexibly parameterize scripts and tailor them to
target specific operating systems and their versions.

## Installation

Stevedore is distributed as a jar, and is available in the
[clojars repository](http://clojars.org/com.palletops/stevedore).

Installation is with lein or your favourite maven repository aware build tool.

### lein project.clj

```clj
:dependencies [[com.palletops/stevedore "0.8.0-beta.7"]]
```

### maven pom.xml

```xml
<dependencies>
  <dependency>
    <groupId>com.palletops</groupId>
    <artifactId>stevedore</artifactId>
    <version>0.8.0-beta.7</version>
  </dependency>
<dependencies>

<repositories>
  <repository>
    <id>clojars</id>
    <url>http://clojars.org/repo</url>
  </repository>
</repositories>
```

## Examples

### Basic Usage

Let's write a basic script; nothing fancy just a plain ole' thing you can
easily write in your scripting language of choice. Very static.

```clojure
(use 'pallet.stevedore)
(require 'pallet.stevedore.bash)

(print
 (with-script-language :pallet.stevedore.bash/bash
   (script
    ("ls" "/some/path")
    (defvar x 1)
    (println @x)
    (defn foo [x] ("ls" @x))
    ("foo" 1)
    (if (= a a)
      (println "Reassuring")
      (println "Ooops"))
    (println "I am" @("whomai")))))
```

Outputs:
```bash
    # form-init7685875326394262653.clj:2
ls /some/path
    # form-init7685875326394262653.clj:3
x=1
    # form-init7685875326394262653.clj:4
echo ${x}
foo() {
x=$1
    # form-init7685875326394262653.clj:5
ls ${x}
}
    # form-init7685875326394262653.clj:6
foo 1
    # form-init7685875326394262653.clj:7
if [ "a" == "a" ]; then echo Reassuring;else echo Ooops;fi
    # form-init7685875326394262653.clj:10
echo I am $(whomai)
```

### Interpolating Clojure

Here is where we start seeing some power. Notice everything escaped with `~`
gets evaluated as clojure code before becoming parts of arguments for `script`.

```clojure
(print
 (with-script-language :pallet.stevedore.bash/bash
   (let [path "/some/path"]
     (script
      ("ls" ~path)
      ("ls" ~(.replace path "some" "other"))))))
```

Outputs:

```bash
    # form-init7685875326394262653.clj:6
ls /some/path
    # form-init7685875326394262653.clj:7
ls /other/path
```

### Generating Scripts

That's cool, but here's one better. Let's do the same thing as the previous
snippet but let's generate the script using a function that takes a path as an
argument. This lets us generate a slightly different script with each different
argument.

```clojure
(defn list-path [path]
  "Replaces the \"some\" portion of path argument with the \"other\" string."
  (script
   ("ls" ~path)
   ("ls" ~(.replace path "some" "other"))))

(print (with-script-language :pallet.stevedore.bash/bash
         (list-path "/some/path")))
```

Outputs:

```bash
    # form-init7685875326394262653.clj:7
ls /some/path
    # form-init7685875326394262653.clj:8
ls /other/path
```

```clojure
(print (with-script-language :pallet.stevedore.bash/bash
         (list-path "/some/different/path")))
```

Outputs:

```bash
    # form-init7685875326394262653.clj:7
ls /some/different/path
    # form-init7685875326394262653.clj:8
ls /other/different/path
```

### Composing Scripts

Concatenate scripts together using `do-script`.

```clojure
(print (with-script-language :pallet.stevedore.bash/bash
		 (do-script
          (list-path "/some/path")
          (list-path "/some/different/path"))))
```

Outputs:

```bash
# form-init7685875326394262653.clj:5
ls /some/path
    # form-init7685875326394262653.clj:6
ls /other/path
# form-init7685875326394262653.clj:5
ls /some/different/path
    # form-init7685875326394262653.clj:6
ls /other/different/path
```

Chain scripts together with `&&` using `chained-script`.

```clojure
(print (with-script-language :pallet.stevedore.bash/bash
		 (chained-script
          (list-path "/some/path")
          (list-path "/some/different/path"))))
```

Outputs:

```bash
# form-init7685875326394262653.clj:4
    # form-init7685875326394262653.clj:5
ls /some/path
    # form-init7685875326394262653.clj:6
ls /other/path && \
# form-init7685875326394262653.clj:5
    # form-init7685875326394262653.clj:5
ls /some/different/path
    # form-init7685875326394262653.clj:6
ls /other/different/path
```

Chain your scripts and exit if the chain fails using `checked-script`.

```clojure
(print (with-script-language :pallet.stevedore.bash/bash
		 (checked-script
          (list-path "/some/path")
          (list-path "/some/different/path"))))
```

Outputs:

```bash
echo '    # form-init7685875326394262653.clj:5
ls /some/path
    # form-init7685875326394262653.clj:6
ls /other/path
...';
{
    # form-init7685875326394262653.clj:5
    # form-init7685875326394262653.clj:5
ls /some/different/path
    # form-init7685875326394262653.clj:6
ls /other/different/path

 } || { echo '#>     # form-init7685875326394262653.clj:5
ls /some/path
    # form-init7685875326394262653.clj:6
ls /other/path
 : FAIL'; exit 1;} >&2 
echo '#>     # form-init7685875326394262653.clj:5
ls /some/path
    # form-init7685875326394262653.clj:6
ls /other/path
 : SUCCESS'
```

## Further Reading

- [More examples and explanations](http://palletops.com/pallet/doc/reference/0.8/script/)
- [API documentation](http://pallet.github.com/stevedore/autodoc/index.html)
- [Annotated source](http://pallet.github.com/stevedore/marginalia/uberdoc.html)
- [More usage examples(tests)](https://github.com/pallet/stevedore/tree/develop/test/pallet/stevedore)

## Support

- [Google Groups](http://groups.google.com/group/pallet-clj)
- [IRC](http://webchat.freenode.net/?channels=pallet)

## License

Licensed under [EPL](http://www.eclipse.org/legal/epl-v10.html)

Copyright 2010, 2011, 2012, 2013 Hugo Duncan.
