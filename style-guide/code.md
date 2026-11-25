# Code: shared principles

These principles hold in every language. The language pages say *where* each one goes; this page says *what* and *why*. Section numbers (§) refer to the [Engineering Manual](https://www.tcl-lang.org/doc/engManual.pdf).

## 1. Every interface says what a caller needs to know, and nothing else (§3.2)

Document public interfaces, including operator-facing scripts, roles and playbooks, and any operation with meaningful side effects in three parts:

- **Summary:** what it does and why you would call it. Write a high-level summary, not a description of the implementation. For callbacks and entry points, also say who calls it and when.
- **Results:** what the caller can see right away: the return value or exit status, output, and anything written back through arguments.
- **Side effects:** what it changes that the caller won't see right away but that affects later behavior. For example, files written, services restarted, units enabled, mounts touched, caches updated, messages sent. Write "None." if there are none.

Side effects matter most in our code. Our scripts restart services, move test data and run as root on hosts that people depend on. A reader should never have to read a function's body to find out whether calling it is safe.

A short private helper with an obvious purpose can use a one-line description when it adds context. For documented interfaces, describe what each argument and exposed data field, config key or inventory variable *means*. Its type is not enough. Where a language has types (type hints, typed parameters), the type goes there and the comment carries only the meaning.

## 2. Write the documentation first (§1, §6.5)

Sketch the required interface contracts before writing the implementation. This helps you decide how the pieces fit together and what callers can rely on. Update the contract as the implementation changes, and include both in the same PR.

## 3. If it is hard to document, it is not clean yet (§6.4)

When a function's documentation is hard to write, the function is usually doing too much. A Side effects section that runs to six items is a design problem, not a documentation problem. Clean code is built on a few simple ideas, and documenting those ideas is enough.

## 4. Don't repeat the code (§6.2)

A comment should tell the reader something the code can't. `# Increment i` is noise. `# Probe the hash table to see if the symbol exists` is useful. Avoid reusing the words in the name: "Map a page" says nothing new about `VmMapPage`, while "Make the given physical page addressable in the kernel's virtual address space" does.

## 5. Document each decision in exactly one place (§6.3)

Put the explanation next to the code that implements the decision: the structure's rules beside the structure, and the function's contract in its docstring. Elsewhere, refer back to it rather than repeating it. Copies drift out of sync.

## 6. Document what has wide impact; keep local things local (§6.1)

Public interfaces, data structures, config keys and globals always need documentation. Local variables in a short function usually don't. In a long function, put a comment before each major block to explain the overall flow.

## 7. Document tricky code, especially code that came from a bug (§6.6)

If correct code depends on something a first-time reader won't know, write that thing down. A bug fix almost always qualifies. Examples from our own systems: systemd never reports `Result=failed`. A timer's next trigger is empty while its service is running. A timeout from `mountpoint` does not mean "not mounted".

## 8. Names (§4.1)

Pick names that convey as much as possible, and challenge them as a devil's advocate would. Use the same name for the same thing everywhere. Don't use two names for one thing, or one name for two things (Sprite lost four months to a variable that sometimes meant a physical disk block and sometimes a logical one). Don't use generic names like `s`, `data` or `tmp` when a specific one exists. Capitalization follows the language community's conventions; see the language pages.

## 9. Small, consistent files

Each file holds one related set of functions and opens with a header comment saying what the file is for. If a file grows past what a reader can hold in their head, split it. The manual suggests 500–2000 lines for C; scripts should usually be much shorter.

## 10. Isolate the system-specific parts (§8)

Put platform differences (Windows versus Linux, network storage versus cloud storage, one instrument vendor versus another) behind one interface, with one implementation per platform. Don't scatter `if host == ...` branches through the code. If you find yourself adding many such branches, the structure needs rethinking.

## Reviewing

A reviewer can reject a change for missing or misleading interface documentation without saying anything else. A reviewer should also ask "what does this change besides its return value?" about any function that runs commands, writes files or talks to a service, and check that the Side effects section answers it.
