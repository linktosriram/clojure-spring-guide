# clojure-spring-guide
Guidelines for using Clojure with Spring Framework

[Clojure Style Guide](https://goo.gl/WWfjB)

# Controllers

* Use [@RestController](https://goo.gl/DpiwnS)
* Use [@GetMapping](https://goo.gl/Y1tX55), [@PostMapping](https://goo.gl/BsajaA) family of annotations over traditional [@RequestMapping](https://goo.gl/naenPG) ones
* Consume and Produce [MediaType.APPLICATION_JSON_UTF8_VALUE](https://goo.gl/US1rUY)
* When returning clojure data structures out of controllers, use [cheshire](https://goo.gl/TCNat)
```clojure
(ns ...
  (:require [cheshire.core :as json]))
```

and then return using

```clojure
(json/generate-string {:a 1 :b 2})
```

* When using annotations like [@PathVariable](https://goo.gl/b5XdMB), [@RequestParam](https://goo.gl/nm7ZiD) etc., specify the `value` attribute as the debugging information is not available. The value attribute can be dash-case.
Otherwise the following error will occur: `Name for argument type [java.lang.String] not available, and parameter name information not found in class file either.`

* We cannot require another namespace or import a class in `(ns :gen-class )`. The code emitted by ns macro calls `:gen-class` before calling any of the `:require` or `:import`.
Thus the required namespace/class or not compiled when `:gen-class` is called. We can use fully qualified names to solve this, but that would be ugly.
A Better solution is to call `(gen-class ...)` after `(ns ...)`

* Method arguments types and return types still need to be fully qualified (except java.lang.*)

* Method names should be camelCase instead of dash-case due to Java restrictions

* If you prefer, you can return [CompletableFuture](https://goo.gl/QQ6uv2) from the controller method to be asynchronous with the following macro

```clojure
(ns ...
  (:import (java.util.concurrent CompletableFuture)
           (java.util.function Supplier)))
           
(defmacro async
  "Wraps the given `body` inside a CompletableFuture."
  [& body]
  `(CompletableFuture/supplyAsync
     (reify
       Supplier
       ~(list 'get '[this] (cons 'do body)))))
```

and then use it like

```clojure
(async
  (println "Async Controller method")
  (+ 3 5))
```

## Complete Example

```clojure
(ns com.clojurespring.controller
  (:require [cheshire.core :as json])
  (:import (org.springframework.web.bind.annotation RestController GetMapping PathVariable)))

(gen-class
  :name ^{RestController {}} com.clojurespring.SampleController
  :methods [[^{GetMapping {:value    ["/hello"]
                           :produces ["application/json;charset=UTF-8"}}
                           hello [^{PathVariable {:value "name"}} String] java.util.concurrent.Future]])

(defn -hello
  [this name]
  (async
    (println "Inside async controller...")
    (json/generate-string {:a       1
                           :message (str "Hello " name " from Clojure controller"))))
```
### Request
```bash
curl -X GET -H "Accept: application/json;charset=UTF-8" "http://server.name/hello/rich"
```

### Response
```json
{
  "a": 1,
  "message": "Hello rich from Clojure controller"
}
```

# Contributing

Nothing written in this guide is set in stone. It's my desire to work together with everyone interested in Clojure coding style, so that we could ultimately create a resource that will be beneficial to the entire Clojure community.

Feel free to open tickets or send pull requests with improvements. Thanks in advance for your help!
