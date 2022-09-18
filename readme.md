# Flag parser
A robust flag parser.

## Usage
```cpp
int main(int argc, char **argv)
{
    using namespace flag;

    Parser parser(std::span{argv, (size_t)argc}, Options{
        .flag_prefix = "--",
        // will fail if an incorrect flag is used
        .strict_flags = true,
    });

    Result result = parser
    .set({
        .name = "b",
        .description = "this is a cool bool flag",
        .data = false,
        .type = Bool,
    })
    .set({
        .name = "num",
        .description = "this flag is a number",
        // will be the default value if no flag is triggered
        .data = 10.f,
        .type = Number,
        .aliases = {"n"}
    })
    .set({
        .name = "string",
        .description = "this is a string",
        .data = "nothing set",
        .type = String,
        .aliases = {"s"}
    })
    .set({
        .name = "help",
        .description = "the help command",
        .data = false, 
        .aliases = {"h"}
    })
    .parse();

    if (!result.ok)
    {
        std::cout << "on flag " << result.flag_id << " error: " << result.error << '\n';
        return 0;
    }

    // an alias can also be used for lookups
    Flag &b_flag = parser.table()["b"]; 

    if (b_flag.triggered)
        std::cout << "b was triggered!\n";

```

## Types 

### Flag 
```cpp
    struct Flag 
    {
        // the name of the flag
        std::string_view name;

        // the flag description
        std::string_view description;

        // the actual data of the flag. its also the default value if set.
        FlagData data{};

        // the type of the flag. by default it will be Any
        Type type = String;

        // the flag aliases
        std::initializer_list<std::string_view> aliases;

        // will be set to true if the flag is ever triggered
        bool triggered = false;

    };
```

### Variant
```cpp
    // the type of a flag
    enum Type : uint8_t 
    {
        String, Number, Bool 
    };

    // the variant used to store flag data
    using FlagData = std::variant<std::string_view, double, bool>;
```

### Util 
```cpp
    struct Options 
    {
        // the prefix used for all flag names
        std::string_view flag_prefix = "-";
        // when set to true the parse function will fail when an incorrect flag is used
        bool strict_flags = true;
    };

    struct Result 
    {
        // if set to true the parser had no errors
        bool ok = true;
        // the flag id that caused the error. likely an incorrect flag id was used
        std::string_view flag_id;
        // the error message
        std::string_view error;
    };
```