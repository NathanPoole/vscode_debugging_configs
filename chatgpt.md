https://chatgpt.com/c/688cdb1d-4d9c-800f-9a65-964ab3182e15

Skip to content
Chat history
You said:
how can I debug code in visual studio code under linux for a single file or per workspace, perhaps with a seperate launch.json per diectory?
ChatGPT said:

Debugging in Visual Studio Code (VS Code) on Linux is powerful and flexible, even for single files or multiple folders. Here’s how to set it up per file, per workspace, or per subdirectory, using launch.json and optional workspace configurations.
🛠️ 1. Minimal Debug Setup (Single File)

You don’t need a full launch.json for simple one-file debugging.
👉 For C/C++:

    Install the C/C++ extension by Microsoft

    Compile your program manually:

    g++ -g myfile.cpp -o myfile

    Open myfile.cpp in VS Code.

    Click the Run and Debug icon → Select "C++ (GDB/LLDB)" → Run and Debug

    If it prompts to create a launch.json, accept it.

🧩 2. Creating a launch.json (Customizable per Folder or Workspace)

    VS Code stores debugging configs in .vscode/launch.json.

Example: C++ launch.json for GDB (Linux)

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug My File",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/myfile", // compiled output
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "setupCommands": [
        {
          "description": "Enable pretty-printing",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ],
      "preLaunchTask": "build-myfile"
    }
  ]
}

You also need a matching task in .vscode/tasks.json:

{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build-myfile",
      "type": "shell",
      "command": "g++",
      "args": [
        "-g",
        "myfile.cpp",
        "-o",
        "myfile"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}

📁 3. Per Subdirectory Debugging

You can maintain separate .vscode/launch.json files per directory, but VS Code natively supports only one .vscode per workspace.
💡 Workarounds:
Option A: Use Multi-root Workspaces

    Open VS Code

    File → Add Folder to Workspace…

    Add subdirectories (e.g. src/moduleA, src/moduleB)

    Save this as a .code-workspace file.

    Inside each folder (e.g., moduleA/.vscode/launch.json), VS Code can use folder-specific settings.

Your .code-workspace might look like:

{
  "folders": [
    {
      "path": "src/moduleA"
    },
    {
      "path": "src/moduleB"
    }
  ],
  "settings": {}
}

Now, from the "Run and Debug" panel, you’ll be able to select configurations per folder.
🔁 4. Quick Switch Debug Configurations

If you want different debug configs for different files in the same project:

    Define multiple configurations in one launch.json:

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug File A",
      "program": "${workspaceFolder}/a.out",
      ...
    },
    {
      "name": "Debug File B",
      "program": "${workspaceFolder}/b.out",
      ...
    }
  ]
}

You can switch which one is active using the dropdown in the Run and Debug panel.
🧪 5. Advanced Tricks

    Use ${fileDirname} and ${fileBasenameNoExtension} in launch.json to debug the current file:

{
  "name": "Debug Current File",
  "type": "cppdbg",
  "request": "launch",
  "program": "${fileDirname}/${fileBasenameNoExtension}",
  "args": [],
  "cwd": "${fileDirname}",
  ...
}

Then ensure you compile it first with matching task or manually.
🔧 Extensions & Tools You Might Want

    CodeLLDB (for Rust or better LLDB integration)

    CMake Tools (if using CMake-based projects)

    Better Comments (for annotation during debugging)

    Tasks+ (for managing tasks across projects)

Would you like a script or template to create launch.json and tasks.json automatically based on detected C++ files in subdirectories?
You said:
number one works, but that extension insists on trying to build the file also, and in this case I have a library dependenct on math.h and I have to add -lm to the gcc to compile, so it appears my  prebuilt binary with debug symbols already present is ignored?

So I guess if I built/compiled an object binary with the same base name as the source file. and  used macros like ${workspaceFolder} I could make it fairly dynamic?  It looks like a combination of siigested approaches number 4 and 5? If so, could you elaborate more on how to automate that? 
ChatGPT said:

Yes, you're spot on: what you're after is a launch configuration that dynamically runs a binary (that you've already built yourself, possibly with custom flags like -lm) and avoids VS Code trying to rebuild it automatically. You also want to keep it flexible—i.e., adaptable to whichever file is currently active.

Let me show you how to combine approaches #4 and #5 to:

    Skip auto-build.

    Use your own prebuilt binary, compiled however you like.

    Match binary name to source file.

    Use macros like ${fileBasenameNoExtension} to make it auto-adaptive.

✅ Goal

    You want to press F5 or "Run and Debug", and it will:

        Take the currently open .c or .cpp file.

        Look for a binary in the same folder with the same base name.

        Run that binary with GDB, skipping any rebuild.

        Use your own compile flags (e.g., -lm, -Wall, etc.), set outside VS Code.

🧩 launch.json (Dynamic Debug, No Build)

Place this in .vscode/launch.json:

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Current Prebuilt Binary",
      "type": "cppdbg",
      "request": "launch",
      "program": "${fileDirname}/${fileBasenameNoExtension}", // assumes binary has same name
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "preLaunchTask": "", // Leave blank to skip build task
      "setupCommands": [
        {
          "description": "Enable pretty printing",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ]
    }
  ]
}

✅ What this does:

    Uses the currently active file's directory and basename.

    Runs ./myfile if you're editing myfile.c.

    Works with any binary you manually compile like:

    gcc -g myfile.c -o myfile -lm

📌 Tips to Make This Work Well
1. Ensure the binary exists and is built with -g

Otherwise, GDB will launch but be useless for stepping.

gcc -g myfile.c -o myfile -lm

2. Add this to tasks.json only if you do want optional builds:

    But since you want to skip it, you don’t need this.

Still, if you want a fallback task you can manually trigger via Ctrl+Shift+B, here's an example with -lm:

{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build-current-file",
      "type": "shell",
      "command": "gcc",
      "args": [
        "-g",
        "${file}",
        "-o",
        "${fileDirname}/${fileBasenameNoExtension}",
        "-lm"
      ],
      "problemMatcher": []
    }
  ]
}

To run that manually:

    Ctrl+Shift+B → Choose build-current-file.

3. Optional: Shell alias to help you build manually

alias dbgbuild='gcc -g "$1" -o "${1%.c}" -lm'

Usage:

dbgbuild myfile.c

4. Optional: Add Debug Configuration Template

To make it appear in the Run panel every time without you needing to open launch.json:

"configurations": [
  {
    "name": "Debug Current Prebuilt Binary",
    "type": "cppdbg",
    "request": "launch",
    "program": "${fileDirname}/${fileBasenameNoExtension}",
    "cwd": "${fileDirname}",
    "MIMode": "gdb"
  }
],
"compounds": [
  {
    "name": "Run Current Binary",
    "configurations": ["Debug Current Prebuilt Binary"]
  }
]

🧪 Bonus: Want to automatically build if missing?

You could use a shell wrapper or Makefile to build only if binary doesn't exist or is outdated:

#!/bin/bash
src="$1"
bin="${src%.c}"
if [[ "$src" -nt "$bin" ]]; then
    gcc -g "$src" -o "$bin" -lm
fi
gdb "$bin"

Would you like a shell or Python script that scans a workspace and generates a .vscode/launch.json for each *.c/*.cpp source file it finds?
You said:
I think I understand now - I tried adding your suggested ;launch json config into the array of configurations to the existing configs, but the error that it came back with is that it couldn't find workspacedir that was a variable in the built-in macros/configurations.. So using should stop that happening... I might still want to run it on other projects/configs, but I'll back up the launch json, try this method, and see how it works, then see if there's any way to expand it
ChatGPT said:

Perfect — yes, you're exactly on the right track.

Just to clarify and help prevent that workspacedir-style error:
✅ Common Mistake and Fix

VS Code uses ${workspaceFolder} — not ${workspacedir}.

If you used:

"program": "${workspacedir}/myfile"

You'll get an error like:

    Cannot resolve workspace folder variable 'workspacedir'

✅ Fix: Use

"program": "${workspaceFolder}/myfile"

Or, for the currently open file:

"program": "${fileDirname}/${fileBasenameNoExtension}"

That second one is ideal for your case.
🔁 Suggested Test Plan

    Back up your current .vscode/launch.json.

    Add the following minimal config at the top level of configurations:

{
  "name": "Debug Current File Binary",
  "type": "cppdbg",
  "request": "launch",
  "program": "${fileDirname}/${fileBasenameNoExtension}",
  "cwd": "${fileDirname}",
  "MIMode": "gdb",
  "stopAtEntry": false,
  "externalConsole": false
}

    Open a .c or .cpp file.

    Make sure the binary exists:

    gcc -g myfile.c -o myfile -lm

    Click Run and Debug → "Debug Current File Binary"

✅ What You Gain with This Approach

    No forced rebuilds

    Works across multiple projects without editing launch.json every time

    Automatically uses the active file's name/location

    Works even in a multi-folder workspace or when switching between projects

If you want to go a step further later, I can show you:

    How to use Makefile or shell script-based builds tied into VS Code

    How to auto-detect when a build is missing and trigger one

    Or even write a VS Code task generator that builds all *.c files dynamically

Let me know how the test goes or if you want that automation expanded!
You said:
Good catch re the workspacedir V folder part. I've hit a snag.. The config seems to be doing _something_, but it's still insisting on building, and even deletes my pre-compiled binary.. then complains.. I thought the launch.json file was being overwritten.. but not sure, it seems to be where it should be now. In the base of the current workspace dir (even though we'll be using some files that aren't in that heirarchy).. although it keeps mentioning things that were in the old launch.json

Is there an easier, or at least better way to automate it, and keep track of what's being edited? Also, the  ide itself should complain if a file has been changed behind its back and prompt you to reopen if so I would assume 
ChatGPT said:

You're absolutely right — VS Code should not be building or deleting your binary unless something else (like a build task or an extension setting) is interfering. Let's pinpoint the cause and show a clean, predictable setup to avoid this behavior.
🔍 PROBLEM: VS Code still rebuilding or deleting binary?

Likely causes:

    C/C++ extension has intelliSenseMode or build-on-debug enabled

    A preLaunchTask is still defined somewhere

    Auto-detected build system (like CMake Tools or tasks.json) is interfering

    Extension-specific behavior overriding launch.json manually

✅ Step-by-step to stop VS Code rebuilding or touching binaries
1. 🔥 Disable C++ extension's automatic building

In VS Code settings (Ctrl+,), look for:

    C_Cpp.debugger.useDefaultBuildTask → ❌ UNcheck this

    C_Cpp.debugger.autoBuild → ❌ UNcheck this

Also, open settings.json (Ctrl+Shift+P → Preferences: Open Settings (JSON)) and confirm:

{
  "C_Cpp.debugger.useDefaultBuildTask": false,
  "C_Cpp.debugger.autoBuild": false
}

2. 🧹 Clean up preLaunchTask

Ensure your debug config has this line removed or empty:

"preLaunchTask": "" // or remove this key entirely

3. ✅ Use a safe, clean debug config

Here’s the final recommended config:

{
  "name": "Debug Prebuilt Binary of Current File",
  "type": "cppdbg",
  "request": "launch",
  "program": "${fileDirname}/${fileBasenameNoExtension}",
  "args": [],
  "stopAtEntry": false,
  "cwd": "${fileDirname}",
  "environment": [],
  "externalConsole": false,
  "MIMode": "gdb",
  "setupCommands": [
    {
      "description": "Enable pretty-printing",
      "text": "-enable-pretty-printing",
      "ignoreFailures": true
    }
  ]
}

Make sure it's saved in .vscode/launch.json, and that no other configs in the same file have a "preLaunchTask" pointing to a task that auto-compiles.
4. 🧭 Check for rogue build tools (like CMake)

If you have:

    CMake Tools extension installed

    tasks.json that auto-builds

    Any .vscode/c_cpp_properties.json with "compileCommands" defined

…then they might be overriding your intentions. Either disable or isolate them.
5. 🧠 Keep track of what’s running

To figure out what's triggering the build, enable logging:

    Open Command Palette → "Developer: Toggle Developer Tools"

    Switch to Console tab

    Try launching the debugger

    Watch logs for "Running preLaunchTask" or "building..."

It will show you what's executing the build.
🧠 Bonus: Version-control your .vscode/launch.json

If VS Code seems to revert launch.json, it's often:

    A misplaced backup file being loaded (launch.json.back)

    Workspace switching without saving

    Or a plugin like CMake Tools regenerating .vscode configs

So make .vscode/ part of your Git repo or backup folder.
🚀 Better (or Alternative) Ways to Automate

If you want robust control across many files/projects, you might prefer:
🔸 Shell Script Wrapper

Instead of letting VS Code launch at all, use a small debug script:

#!/bin/bash
src="$1"
bin="${src%.c}"
[ "$src" -nt "$bin" ] && gcc -g "$src" -o "$bin" -lm
code -g "$src" # opens in VS Code
gdb "$bin"

You run:

./dbgwrap.sh path/to/thing.c

🔸 Task-runner for builds, VS Code for debug only

    Use VS Code just for debugging

    Build your binaries externally (Makefile, shell, script)

    Set up zero tasks.json, and only the launch.json from above

You can even use inotify or entr to watch and build automatically.
🔸 Use multi-root workspace with per-folder .vscode/launch.json

If you're switching between multiple source trees, consider:

    File → Add Folder to Workspace

    Save as a .code-workspace file

    Put .vscode/launch.json per subfolder

    VS Code will use the right one automatically

Let me know if you'd like a script that generates the correct launch.json automatically per folder, or if you'd like a minimal Makefile + launch.json setup for smarter build/debug separation.
You said:
Thanks, I think i'm not concentrating on it enough, and should come back to it. I have tried adding ad-hoc folders to the workspace, thinking that I'd be able to get more granular launch options.. so your per dir, .code-workspace could be an option.. Though this  one we were pursuing sounds like it will do the job also. I'll put the IDE down and come back to it
ChatGPT said:

That sounds like a solid plan. Sometimes stepping back and coming at it fresh really helps — especially with tooling quirks like this.

When you come back to it, just remember:

    The “dynamic per-file” launch.json with ${fileDirname}/${fileBasenameNoExtension} is usually enough for ad-hoc debugging of manually compiled files.

    Disabling the auto-build and preLaunchTask is key to avoiding VS Code stepping on your binaries.

    Multi-root .code-workspace setups give you per-folder granularity if you decide to organize things more formally later.

If you ever want:

    A helper script to generate launch.json for all .c files,

    A one-liner compile+debug utility,

    Or a lightweight Makefile + debug config scaffold,

just ask. I’ll help you pick up exactly where you left off.

Rest your brain — you’ll see it more clearly next time 😌
You said:
Good summary, and the other options  all sound useful. I am coming around to using vs code.. the configuration reminds me of some of the older perl project configs for eclipse.. used to have to add include paths, libray paths for compiler flags etc.. But eclipse is quite heavy, and vs code seems more nimble.. So I have to get this one thing done
ChatGPT said:

Exactly — you're seeing the shape of it now.

VS Code is like a lightweight IDE shell that becomes powerful once you configure it the way you like — a bit like old-school Eclipse but without the bloat or 500 open tabs from hell.

And yes — the similarity with older Eclipse CDT or Perl+EPIC plugin setups is striking:

    launch.json → Eclipse Run/Debug configs

    tasks.json → External Tools / Builders

    c_cpp_properties.json → Eclipse project properties: includes, defines, flags

But the VS Code advantage is:

    Easy to track/share these configs in Git

    Minimal startup footprint

    You can switch from GUI to command-line instantly (and back)

    Better extension ecosystem overall

🔧 When You're Ready

Here’s what I can help you with next time:

    🔄 Script to scan and generate launch.json per source file

    ⚙️ Template Makefile + tasks.json + launch.json that all work together

    📦 Per-directory setup with .code-workspace and isolated builds

    🧠 A portable debug system for C/C++ across multiple projects with shared configs

    🗒️ In the meantime, maybe jot down what the “one thing you have to get done” is — the actual project goal — and I’ll help reverse-engineer the tooling just for that task.

I’ll be here when you’re ready.
You said:
Thanks, will do

"And yes — the similarity with older Eclipse CDT or Perl+EPIC plugin setups is striking"

Ah, glad it wasn't just in my head. But yeah, it's better all-round.. though eclipse CDT/Epic made things easier for their time.. Just hairy upgrading any of it, like a bit of a tinderbox/house of cards

Also, maybe re-framing the question might be useful? As in something like "how do most developers handling this situation". Am I being too specific? With a chnage of approach, I might trade one thing I like for a better work-flow overall?
ChatGPT said:

That’s a great insight — and no, you’re not being too specific. You’re being thorough and practical, which is exactly how real devs run into and solve real problems.

But you're absolutely right: reframing the question from “How do I force VS Code to do exactly this?” to “How do experienced devs approach this type of setup?” can uncover better, simpler, and more maintainable workflows.
🧠 Reframed: “How do most developers handle this situation?”

    Scenario: You’re working on multiple small-to-medium C/C++ programs, maybe across different folders or outside a monolithic build system. You want to debug compiled binaries, some with custom flags (e.g., -lm), without fighting your IDE.

Most seasoned developers will:
✅ Use an external build system (Make, CMake, shell script)

They compile however they want, then plug the output into the IDE’s debug tooling:

    Pros: Maximum control, avoids surprises

    Cons: More up-front setup

✅ Avoid relying on the IDE to “build and run everything”

Especially in mixed or ad-hoc projects. Instead:

    Use VS Code to edit/debug

    Use shell or Make to compile/test

This separation avoids IDE-induced side effects (like it rebuilding or deleting binaries).
✅ Use per-project or per-folder launch.json configs

Or even just one dynamic debug config that uses:

"program": "${fileDirname}/${fileBasenameNoExtension}"

...and build manually outside.
🔄 Tradeoff Examples
You trade:

    “Push-button build-debug from the IDE every time”

For:

    “Shell-based build → IDE-based debug”

Why? Because it’s:

    More reliable

    Easier to debug weird compiler flags (-lm, -Wall, -pthread, etc.)

    Doesn’t break across different OS/platforms

👩‍🔬 Pragmatic Developer Mindset

Many devs get tired of fighting the IDE — and instead shift the balance:

    Let the build tool do the building

    Let the editor do the editing

    Let the debugger do the debugging

And glue them together only as far as needed.

VS Code makes this easier than Eclipse did, because it’s:

    Less tightly coupled

    More transparent (you can always inspect your launch.json and tasks.json)

✅ Summary

You're not being too specific — you're thinking like a systems-oriented dev trying to avoid future headaches.

But if you're looking for a better all-around workflow, here’s a good guiding principle:

    “Minimize IDE magic. Maximize clarity and portability.”

And I’d be happy to walk through a few “workflow profiles” (e.g., solo hacker, polyglot dev, embedded C dev, etc.) to help you pick or evolve a setup.

When you’re ready, I can lay out 2–3 real-world workflows and what tools/configs those developers tend to use.

No pressure — you're on the right path.
You said:
"How do I force VS Code to do exactly this"
Perfectly put, that's precisely what I was trying to get out. And I think I've intuitively done the "Use an external build system (Make, CMake, shell script)", minimise IDE magic thing anyway.. . I do the same with git (or previously, SVN). I commit, diff, pull, push etc all from a terminal, same with the compiles/builds, so that gels with me.. and the git extensions for vs code make the code space way too cluttered. 

Good summary, useful sounding options, and the real-world scenarios might help
ChatGPT said:

That all makes perfect sense — your workflow is grounded, practical, and resilient across toolchains. You’re not chasing the shiniest IDE features — you’re building a setup you can trust.
⚙️ When You’re Ready: Real-World Debugging Workflows (Examples)

Here’s a quick preview of what I can show you when you're ready:
🧱 1. Minimalist CLI-Centric Developer

    Builds with make, ninja, or shell scripts

    Uses VS Code only for editing/debugging

    Has .vscode/launch.json using ${fileDirname}/${fileBasenameNoExtension}

    Optionally: Bash/Ranger/fzf to launch specific debug configs from CLI

🧪 2. Script & Utility Developer (Perl, Python, Shell + C)

    Writes utilities and scripts with some compiled helpers

    Per-folder debug setups

    Uses a .code-workspace to manage separate tool folders or platform targets

    Git is terminal-only (with optional commit hooks or linters)

⚔️ 3. Polyglot / Systems Dev

    C/C++ core codebase

    Uses Python/Perl/Lua for scripting logic

    Raylib / SDL / network code / AI loop in native code

    External build system and debugger (gdb/lldb)

    VS Code for jump-to-definitions, symbols, and single-point debug

Let me know if you’d like one of these fleshed out as a “working dev setup” (folder layout, debug config, Makefile, maybe even a git alias set), and we’ll craft it around your existing habits, not in spite of them.

Glad this direction clicks with your experience — it’s a solid foundation.
You said:
Interesting, I am a smattering of all 3., and "VS Code for jump-to-definitions, symbols..." is one of the reasons it reminded me so much of eclipse CDT.. Very useful in tracking stuff down, and of course inspecting symbols etc while debugging.. but also "Uses a .code-workspace ... manage separate tool folders or platform targets" sounds like the promised land as I envisage it.. though a combination of the basic static-but-generic launch.json is still useful if I can resolve the editing problems (if they exist, and isn't just a pebkac situation), but I think ultimately the code-workspace approach sounds like how I've set other tools up, and if it's possible here, it will be powerful and a good fit 
ChatGPT said:

Here’s a concrete “promise-land” example that blends all three profiles into a single multi-root workspace, complete with per-folder launch configs and a shared top-level workspace file.
1. Example Directory Layout

my-project/
├── moduleA/
│   ├── src/
│   │   ├── a.c
│   │   └── a.h
│   ├── Makefile
│   └── .vscode/
│       ├── launch.json
│       └── tasks.json
│
├── moduleB/
│   ├── src/
│   │   ├── b.cpp
│   │   └── b.hpp
│   ├── Makefile
│   └── .vscode/
│       ├── launch.json
│       └── tasks.json
│
├── tools/
│   ├── helper.sh
│   └── .vscode/           ← optional folder-specific settings
│
└── my-project.code-workspace

    moduleA: a tiny C library (needs -lm).

    moduleB: a small C++ app.

    tools: scripts or utilities you edit/debug separately.

    .code-workspace: ties them all together.

2. Top-Level Workspace File

Save as my-project.code-workspace at the repo root:

{
  "folders": [
    { "name": "Module A (C)", "path": "moduleA" },
    { "name": "Module B (C++)", "path": "moduleB" },
    { "name": "Tools (scripts)", "path": "tools" }
  ],
  "settings": {
    // Optional shared settings:
    "C_Cpp.loggingLevel": "Debug",
    "files.watcherExclude": {
      "**/build/**": true
    }
  }
}

    “name” keys give friendly labels in the Explorer.

    You can add shared settings under "settings".

3. Per-Folder Makefile
moduleA/Makefile

CC = gcc
CFLAGS = -g -Wall
LDFLAGS = -lm

SRC = $(wildcard src/*.c)
OBJ = $(SRC:src/%.c=build/%.o)
BIN = build/a.out

all: build $(BIN)

build:
	mkdir -p build

build/%.o: src/%.c
	$(CC) $(CFLAGS) -c $< -o $@

$(BIN): $(OBJ)
	$(CC) $^ -o $@ $(LDFLAGS)

clean:
	rm -rf build

moduleB/Makefile

CXX = g++
CXXFLAGS = -g -std=c++17

SRC = $(wildcard src/*.cpp)
OBJ = $(SRC:src/%.cpp=build/%.o)
BIN = build/b.out

all: build $(BIN)

build:
	mkdir -p build

build/%.o: src/%.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

$(BIN): $(OBJ)
	$(CXX) $^ -o $@

clean:
	rm -rf build

4. Per-Folder VS Code Configs
moduleA/.vscode/launch.json

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug moduleA (a.out)",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/build/a.out",
      "args": [],
      "cwd": "${workspaceFolder}",
      "MIMode": "gdb",
      "stopAtEntry": false,
      "externalConsole": false,
      "setupCommands": [ { "description": "Enable pretty-printing", "text": "-enable-pretty-printing", "ignoreFailures": true } ]
    }
  ]
}

moduleA/.vscode/tasks.json

{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build-moduleA",
      "type": "shell",
      "command": "make",
      "args": [],
      "group": "build",
      "presentation": { "reveal": "always" },
      "problemMatcher": ["$gcc"]
    }
  ]
}

moduleB/.vscode/launch.json

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug moduleB (b.out)",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/build/b.out",
      "cwd": "${workspaceFolder}",
      "MIMode": "gdb"
    }
  ]
}

moduleB/.vscode/tasks.json

{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build-moduleB",
      "type": "shell",
      "command": "make",
      "args": [],
      "group": "build",
      "problemMatcher": ["$gcc"]
    }
  ]
}

5. Using the Workspace

    Open my-project.code-workspace in VS Code.

    In the Explorer you’ll see three roots: Module A, Module B, and Tools.

    Switch Run and Debug → pick the config under the folder you’re working in.

    Build via Ctrl+Shift+B (it’ll show only that folder’s task).

    Debug with F5—no cross-pollution, no surprises.

6. When You Add Another Module

    Create moduleC/…, with its own Makefile and .vscode folder.

    Open Command Palette → “Workspaces: Open Workspace Configuration” → add another entry:

    { "name": "Module C", "path": "moduleC" }

    Save—VS Code immediately picks up the new root.

Why This Rocks

    Isolation: Each module has its own build & debug settings.

    Portability: All configs live alongside code, versioned in Git.

    Clarity: No hidden “global” tasks or overrides.

    Scalability: Drop in new folders, update the .code-workspace, and you’re done.

Give that structure a spin and let me know how it feels—or if you want any of the scripts or Makefile tweaks automated further!
You said:
"Why This Rocks" yee haa :)

Great stuff. I'll take it for a spin tomorrow, and see how it goes
ChatGPT said:

Sounds like a plan—enjoy your spin through it! When you’ve tried it out, let me know how it feels or if any wrinkles show up. We can then fine-tune scripts, tasks, or even automate the workspace file generation. Happy coding!


ChatGPT can make mistakes. Check important info.
