# Guide: Creating a C++ Extension for Falkon Browser

This guide walks through creating a basic "Hello World" plugin in falkon. 

---

## 1. Project Structure

Create a dedicated folder inside the Falkon source tree. Case sensitivity is critical here for Linux.

**Path:** `src/plugins/HelloWorld/`

- `helloworld.json` — Plugin Metadata
    
- `HelloWorld.h` — Header (Interface)
    
- `HelloWorld.cpp` — Implementation (Logic)
    
- `CMakeLists.txt` — Build Instructions
    

---

## 2. Plugin Metadata (`helloworld.json`)

This file defines how the plugin appears in the Falkon UI.

JSON

```json
{
    "Name": "Hello World Extension",
    "Description": "A simple C++ plugin that greets the user.",
    "Author": "Author_name",
    "Version": "1.0",
    "Icon": "preferences-desktop-notification"
}
```

---

## 3. The Header (`HelloWorld.h`)

The plugin must inherit from `PluginInterface`. The `Q_PLUGIN_METADATA` macro links this header to your JSON file.


CPP 

```cpp
#ifndef HELLOWORLD_H
#define HELLOWORLD_H

#include "plugininterface.h"
#include <QObject>

class HelloWorld : public QObject, public PluginInterface
{
    Q_OBJECT
    Q_INTERFACES(PluginInterface)
    Q_PLUGIN_METADATA(IID "org.falkon.HelloWorld" FILE "helloworld.json")

public:
    explicit HelloWorld();

    // Mandatory interface methods
    void init(InitState state, const QString &settingsPath) override;
    void unload() override;
    bool testPlugin() override;
    
    // UI methods
    void showSettings(QWidget *parent) override;
};

#endif // HELLOWORLD_H
```

---

## 4. The Implementation (`HelloWorld.cpp`)

**Note:** In Qt6, you must wrap strings in `QStringLiteral()` to avoid the "deleted function" compiler error.

CPP

```cpp
#include "HelloWorld.h"
#include <QMessageBox>
#include <QDebug>

HelloWorld::HelloWorld() : QObject() {}

void HelloWorld::init(InitState state, const QString &settingsPath)
{
    // Suppress unused parameter warnings
    Q_UNUSED(state)
    Q_UNUSED(settingsPath)

    qDebug() << "Hello World Extension: Successfully loaded!";

    // Modern Qt6 requires explicit QString conversion
    QMessageBox::information(nullptr, 
                             QStringLiteral("Falkon Greeting"), 
                             QStringLiteral("Hello World! Your C++ extension is now active."));
}

void HelloWorld::unload()
{
    qDebug() << "Hello World Extension: Unloaded.";
}

bool HelloWorld::testPlugin()
{
    return true; // Tells Falkon the plugin is compatible
}

void HelloWorld::showSettings(QWidget *parent)
{
    QMessageBox::information(parent, 
                             QStringLiteral("Settings"), 
                             QStringLiteral("There are no settings for this simple demo."));
}
```

---

## 5. Build Instructions (`CMakeLists.txt`)

Create this file **inside** the `src/plugins/HelloWorld/` directory.

CMake

```txt
# Define the plugin as a MODULE (Shared Object)
add_library(HelloWorld MODULE
    HelloWorld.cpp
    HelloWorld.h
)   

# Link against the Falkon core library
target_link_libraries(HelloWorld
    PRIVATE
    FalkonPrivate
)

# Include Falkon headers (required for plugininterface.h)
target_include_directories(HelloWorld PRIVATE 
    ${FALKON_LIB_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}
)

# Set the installation path
install(TARGETS HelloWorld DESTINATION ${FALKON_INSTALL_PLUGINDIR})
```

---

## 6. Build and Activation

### Step A: Register the Subdirectory

Open the parent file: `src/plugins/CMakeLists.txt` and add:

CMake

```
add_subdirectory(HelloWorld)
```

### Step B: Compile

Navigate to your `build` directory. Use the `-j` flag to utilize all your CPU cores for a faster build.

Bash

```
cd build
cmake ..
make -j$(nproc) HelloWorld
```

### Step C: Launch

Use the prefix script to ensure Falkon loads your local build instead of the system version.

Bash

```
source prefix.sh
./bin/falkon
```

### Step D: Enable

1. Go to **Preferences > Extensions**.
    
2. Find **"Hello World Extension"**.
    
3. Check the box. Your popup will appear immediately!

![Falkon Hello World Extension](../../assets/helloworld.png)