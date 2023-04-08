## Immunity Debugger

### Interface

#### Shortcuts

| Shortcut | Function                                                     |
| -------- | ------------------------------------------------------------ |
| ALT-C    | CPU Info                                                     |
| ALT-M    | Memory map                                                   |
| ALT-L    | Log data                                                     |
| ALT-B    | Breakpoints                                                  |
| CTRL+F2  | Restart program                                              |
| CTRL+F9  | Execute until return (Finishes out the current function and returns to the calling function) |
| F2       | Set a breakpoint on a highlighted point of code (highlights the address with light blue) |
| F7       | Step into function (Follow the flow)                         |
| F8       | Step over a function (Skip the flow)                         |
| F9       | Debug -> Run (runs until it hits a breakpoint)               |

Double-click any of the memory addresses (in any pane with memory addresses) to show relative positions (in hex) from position that was double-clicked.

#### CPU Screen

- Upper left window - Assembly instructions that make up the application
  - Instruction highlighted in blue is the NEXT instruction to be executed
- Upper right window - Shows CPU Registers and their values. ESP & EIP are the most important ones here.
- Lower left window - Contents of memory at any given address
- Lower right window - Shows the stack and its contents in both hex (DWORD) and ASCII

### Workflow

- Identify the area of the program you want to get to
- Search for text strings found in the code
  - In the Assembly Instructions, right click and choose 'Search for -> All referenced text strings'
  - Double-click the match you want to go to that point in the assembly code
  - 

