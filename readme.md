# Flag parser
A robust flag parser.

## Usage
```cpp
#include <iostream>
#include <variant>

#include "flags.hpp"

using namespace flag;

Result b_fn(Flag &flag)
{
    std::cout << flag.name << " was triggered!\n";
    return {};
}

void report_result(Result result)
{
    std::cout << "error on flag " << result.flag_id << " error: " << result.error << '\n';
}

int main(int argc, char **argv)
{

    Parser parser(std::span{argv, (size_t)argc}, Options{
        .flag_prefix = "--",
        // the string that separates the flag name and value
        .separator = ":",
        // makes it so that the parser will fail if an unknown flag is used
        .strict_flags = true,
    });

    Result result = parser
    .set({
        .name = "b",
        .description = "this is a cool bool flag",
        .data = false,
        .type = Bool,
        // will be called if the flag was triggered
        .fn = b_fn,
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
        report_result(result);
        return 0;
    }

    // must be called to call the flag functions
    result = parser.call();

    if (!result.ok)
        report_result(result);
}
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
        
        // the function that will be called when the flag is triggered. must call Parser::call.
        FlagFn fn = nullptr;
        
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
        // the characters that will be used to separate the flag name and value
        std::string_view separator = "=";
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