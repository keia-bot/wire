# 🧵 Keia &bull; Wire

> Currently working on getting an open-source release underway!  
> Join our [**Discord Server**](https://discord.gg/8R4d8RydT4) for more information.

An i18n-first interaction & message command framework for [**Kord**](https://github.com/kordlib/kord).

## 💾 Example

### Setup

```kt
object MyI18nProvider : I18nProvider {
    override fun getStrings(specifier: String): Map<String, Locale> = TODO()

    override fun getString(specifier: String, locale: Locale): String = TODO()

    override fun interpolate(format: String, locale: String, arguments: Map<String, Any?>): String
}

suspend fun main() {
  val kord = Kord()

  // the Kord#wire extension automatically creates event listeners
  val wire = kord.wire {
    i18n = MyI18nProvider

    // enable support for interaction-based commands
    interactionCommands()

    // enable support for message-based commands
    messageCommands {
      // this disables the message content intent and requires
      // that a mention be used as the prefix.
      mentionOnly = true

      //
      providePrefixes { ctx ->
        ctx.guildOrNull
          ?.let { findGuildPrefixes(it.id) } 
          ?: listOf("!")
      }
    }
  }

  // add our commands
  wire.add(MyCommand)

  // register our commands with Discord (only applicable to interaction commands.)
  wire.register()

  // login to discord.
  kord.login {
    val ourIntents = Intent.Guilds
    intents = wire.intents + ourIntents
  }
}
```

### Commands

#### 1. Create a command

As chat-input commands are forbidden to have a base-command (`/my-command`) along side sub-commands (`/my-command sub-command`),
Wire has two separate root-command types, `Single` and `Nested`. Due to this fact, message-based commands must also suffer from
this issue.

```kt
object MyCommand : Command.Single() {
  override suspend fun execute(ctx: FinalCommandContext) {
    ctx.respondEmbed("Hi ${ctx.user.mention}!")    
  }
}
```

#### 2. Using Parameters

Wire has a very simple parameter api that allows for them to be easily shared between interactions and messages.

```kt
object MyCommand : Command.Single() {
  enum class Fizz {
    Fizz,
    Buzz,
    FizzBuzz
  }

  // string values with min/max length
  private val P_MESSAGE = p.text("message") {
    min = 5
    max = 30
  }

  // optional values
  private val P_QUOTED = p.optional(p.bool("quoted"))

  // provide completions through static entries or a callback.
  private val P_FOOBAR = p.text("test") {
    // static choices
    choices("foo" to "bar", "bar" to "foo")

    // callback choices
    autocomplete { ctx, term ->
      mapOf("foo" to "bar", "bar" to "foo", "something" to term)
    } 
  }

  // enum values using their name (enumName) or ordinal (enumOrdinal)
  // (this is a custom parameter type!)
  private val P_FIZZ_BUZZ = p.enumOrdinal<Fizz>("fizzbuzz")

  override val parameters get() = setOf(P_MESSAGE, P_QUOTED, P_FOOBAR, P_FIZZ_BUZZ)

  override suspend fun execute(ctx: FinalCommandContext) {
    val quoted = ctx.args[P_QUOTED] ?: false
    val message = ctx.args[P_MESSAGE]
    ctx.respondEmbed(if (quoted) "'$message'" else message)
  }
}
```

#### 3. Creating Parameter Types

Wire supports creating custom parameter types using a base type and transform function.

```kt
@JvmInline
value class Percentage(val value: Float)

// A custom type that converts a `text` parameter to a Percentage object.
// The `InvalidParameterException` type can be thrown to report validation issues to the user.
val PercentageType = ParameterType.Custom(
  { value -> Percentage(value.removeSuffix("%").toFloat()) },
  PT.text()
);
```

---

Have any questions? Join our [**Discord Server**](https://discord.gg/8R4d8RydT4)!

<sub>dimensional fun &copy; 2024</sub>
