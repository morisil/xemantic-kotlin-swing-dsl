# xemantic-kotlin-swing-dsl

_Express your Swing code easily in Kotlin_

## Why?

Kotlin provides incredible language sugar over pure Java. Using Kotlin
instead of Java for writing Swing UI already makes the code more concise, but
**what if the Swing was written from scratch, to provide Kotlin-idiomatic way of doing things?**
This is the intent behind the `xemantic-kotlin-swing-dsl` library - to deliver a
[Domain Specific Language](https://en.wikipedia.org/wiki/Domain-specific_language)
for building Swing based UI and react to UI events by utilizing
[Kotlin coroutines](https://kotlinlang.org/docs/coroutines-overview.html). 

## Usage

Add to your `build.gradle.kts`

```kotlin
dependencies {
  implementation("com.xemantic.kotlin:xemantic-kotlin-swing-dsl-core:$libVersion")
  runtimeOnly("org.jetbrains.kotlinx:kotlinx-coroutines-swing:$coroutinesVersion")
}
```

## Example

Here is a simple internet browser. For the sake of example, instead of rendering
the full HTML, it will just download a content from provided URL address
and display it as a text.

```kotlin
import com.xemantic.kotlin.swing.*
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import java.awt.Dimension
import java.net.URI

fun main() = MainWindow("My Browser") {
  val urlBox = TextField()
  val goButton = Button("Go!") { isEnabled = false }
  val contentBox = TextArea {
    preferredSize = Dimension(300, 300)
  }

  urlBox.textChanges.listen { url ->
    goButton.isEnabled = url.isNotBlank()
  }

  merge(
    goButton.actionEvents,
    urlBox.actionEvents
  )
    .filter { goButton.isEnabled }
    .onEach { goButton.isEnabled = false }
    .map { urlBox.text }
    .flowOn(Dispatchers.Main)
    .map {
      try {
        URI(it).toURL().readText()
      } catch (e : Exception) {
        e.message
      }
    }
    .flowOn(Dispatchers.IO)
    .listen {
      contentBox.text = it
      goButton.isEnabled = true
    }

  Border.empty(4) {
    BorderPanel {
      layout {
        hgap = 4
        vgap = 4
      }
      north {
        BorderPanel {
          layout {
            hgap = 4
            vgap = 4
          }
          west {
            Label("URL") }
          center { urlBox }
          east { goButton }
        }
      }
      center { ScrollPane { contentBox } }
    }
  }

}
```

When run it will produce:

![example app image](docs/xemantic-kotlin-swing-dsl-example.png)

### Notable conventions

* No `J` prefix in UI component names (historically `J` was added to differentiate
  Swing components from AWT components and is mostly irrelevant for modern purposes).
* Master JFrame is created with the `WainWindow` builder, which also takes care of setting
  up the `SwingScope` holding a coroutine scope bound to Swing's event dispatcher thread.
* Instead of callbacks, events are delivered through `Flow`s, the `listen()` function will
  collect the flow in the newly launched coroutine, which is cancelled when the window is closed.
* Other coroutine dispatchers, like `IO`, can be used in the event processing pipeline
  by adding the `flowOn`. This makes the cumbersome `SwingWorker` obsolete.
* Each UI component can be immediately configured with direct access to its properties.
* Panels are specified with special builders, allowing intuitive development of the
  component tree.

### Key benefits

* compact UI code with minimal verbosity
* declarative instead of imperative UI building
* reactive event handling using `Flow`s, 

And here is an equivalent code in Java for the sake of comparison:

The code above is taken from the
[MyBrowserKotlin](demo/simple-kotlin-dsl/src/main/kotlin/MyBrowserKotlin.kt) demo.

### How would it look in pure Java Swing?

For the sake of comparison the
[MyBrowserKotlin](demo/simple-java/src/main/java/com/xemantic/kotlin/swing/demo/MyBrowserJava.java)
demo implements exactly the same "browser" in pure Java.

## Other examples

The [demo](demo) folder contains additional examples.
