# Jsoniter Scala 

[![build status](https://travis-ci.org/plokhotnyuk/jsoniter-scala.svg?branch=master)](https://travis-ci.org/plokhotnyuk/jsoniter-scala) [![code coverage](https://codecov.io/gh/plokhotnyuk/jsoniter-scala/branch/master/graph/badge.svg)](https://codecov.io/gh/plokhotnyuk/jsoniter-scala) [![license](http://img.shields.io/:license-Apache%202-green.svg)](http://www.apache.org/licenses/LICENSE-2.0.txt)

Scala macros that generates codecs for case classes, standard types and collections
to get maximum performance of JSON parsing & serialization.

## Features and limitations
- JSON parsing from `Array[Byte]` or `java.io.InputStream`
- JSON serialization to `Array[Byte]` or `java.io.OutputStream`
- Support of UTF-8 encoding
- Parsing of strings with escaped characters for JSON field names, keys & values 
- Configurable serialization of strings with escaped Unicode characters to be ASCII compatible
- Configurable indenting of output
- Case classes should be defined as a top-level class or directly inside of another class or object
- Primitives, boxed primitives, enums, `String`, `BigInt`, `BigDecimal`, `Option`, Scala collections, arrays and value classes for all them or case classes themself are supported as case class fields 
- Types that supported as map keys are all which can be represented by single value, basically it is all above excluding case classes, arrays & collections  
- Fields with default values that defined in a constructor are optional, other fields are required (no special annotation required)
- Fields with default values, empty options & empty collections/arrays are not serialized to provide sparse output 
- Fields can be annotated as transient or just not defined in constructor to avoid parsing and serializing at all 
- Need to materialize for all case classes used in nested structures using implicitly resolvable encoders/decoders
- Key names can be overridden by field annotation
- Field names are matched by hash and can have collisions with other keys on the same JSON object value
- TODO: generate efficient and no-allocation collision resolving for field name matching
- TODO: generate encoders/decoders recursively from top-level type
- TODO: support on top-level for values, collections & arrays to avoid wrapping to redundant case classes
- TODO: implicitly resolvable mappers for ADTs, abstract classes & traits
- TODO: efficient specialized immutable list & set for primitive types

## How to use

Build and publish locally for both Scala versions (release to Sonatype repo is coming)

```sh
sbt clean +publishLocal
```

Add the library to your dependencies list

```sbt
libraryDependencies += "com.github.plokhotnyuk.jsoniter-scala" %% "macros" % "0.1-SNAPSHOT"
```

Generate some serializers for your case classes
    
```scala
import com.github.plokhotnyuk.jsoniter_scala.JsonCodec._

case class Device(id: Int, model: String)
implicit val deviceCodec = materialize[Device]

case class User(name: String, devices: Seq[Device])
val userCodec = materialize[User]
```

That's it! You have generated an instance of `com.github.plokhotnyuk.jsoniter_scala.Codec` which implements both
`com.jsoniter.spi.Decoder` and `com.jsoniter.spi.Encoder` interfaces for your `User`.

Now you can use it:

```scala
import com.github.plokhotnyuk.jsoniter_scala.JsonReader._
import com.github.plokhotnyuk.jsoniter_scala.JsonWriter._

read(userCodec, """{"name":"John","devices":[{"id":1,model:"HTC One X"}]}""".getBytes("UTF-8"))
write(userCodec, User(name = "John", devices = Seq(Device(id = 2, model = "iPhone X"))))
```

To see generated code add the following line to your sbt build file

```sbt
scalaOptions += "-Xmacro-settings:print-codecs"
```

For more features & examples, please, check out
[JsonCodecSpec](https://github.com/plokhotnyuk/jsoniter-scala/blob/master/macros/src/test/scala/com/github/plokhotnyuk/jsoniter_scala/JsonCodecSpec.scala)


## How to develop

Feel free to ask questions by opening issues (Gitter chat is coming), or contribute by creating pull requests (code or/and tests are highly appreciated)

Run tests and check coverage

```sh
sbt clean +coverage +test +coverageReport
```

Run benchmarks

```sh
sbt -no-colors clean 'benchmark/jmh:run -prof gc .*JsonCodecBenchmark.*' >results.txt
```

Current results for the following environment:

[./results.txt](https://github.com/plokhotnyuk/jsoniter-scala/blob/master/results.txt) Intel(R) Core(TM) i7-2760QM CPU @ 2.40GHz (max 3.50GHz), RAM 16Gb DDR3-1600, Ubuntu 15.04, Linux 4.4.0-38-generic, Oracle JDK build 1.8.0_152-b16 64-bit

[./results-5257U.txt](https://github.com/plokhotnyuk/jsoniter-scala/blob/master/results-5257U.txt) Intel(R) Core(TM) i5-5257U CPU @ 2.70GHz (max 3.10GHz, 2 cores, 4 TH), RAM 16Gb DDR3-1867, macos 10.11.6, Oracle JDK build 1.8.0_152-b16 64-bit


## Acknowledges

[Jsoniter Java](https://github.com/json-iterator/java)

[Kryo Macros](https://github.com/evolution-gaming/kryo-macros)

[AVSystem Commons Library for Scala](https://github.com/AVSystem/scala-commons)