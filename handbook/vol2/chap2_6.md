# Chapter 2.6: Introduction to Assembly

In earlier chapters, we have avoided having to look at **assembly code** as it can be quite hard to grasp. However, an unfortunate reality is that in order to be an independent GD modder, **you need to understand assembly**, or at the very least the basics of it. So, without further ado, let's delve into some **x86**!

```cpp
int addTwo(int a, int b) {
    return a + b;
}

int main() {
    int z = addTwo(5, 7);
}
```
Here we have a very simple program in C++: adding two numbers together. Let's compile this to **x86 assembly** and see what we get!
```cpp
0x432b40:
    mov eax, ecx
    add eax, edx
    ret 0

_main:
    mov ecx, 5
    mov edx, 7
    call 0x432b40
    mov ecx, eax
    xor eax, eax
    ret 0
```

If you have never seen assembly code before, this will probably look completely alien to you. It might leave you scratching your head and asking many questions, from what happened to our variables to where the function name went. Let's see if we can understand it better by turning it into **C++-like pseudo-code**:
```cpp
void 0x432b40() {
    eax = ecx;
    eax += edx;
    return;
}

void _main() {
    ecx = 5;
    edx = 7;
    0x432b40();
    ecx = eax;
}
```

The first thing to notice is that after compilation, each function is given a **memory address**; that is, an address within the binary code where the function can be found. The second thing is that **in binary code, there are no names**. There are only memory addresses, **instructions**, and **registers**. Functions technically do not even exist in binary code; in there, they are called **subroutines**, and are referenced by their memory address.

The second thing to notice is that our variables have turned into **register assignments**. Registers are like **global variables** in C++; they can be assigned to from anywhere. The actual registers that may appear in the assembly code are **predefined by the assembly language being used**. In **x86**, some of them include `ecx`, `edx`, `epx`, and `xmm0`.

The memory addresses of functions are **not constant** between different compilations; what this means in practice for GD mods is that **a mod that works for GD version 2.113 does not work for version 2.112 or 2.2**, as the memory addresses of the functions are completely different. However, they are constant within copies of the **same compiled binary**; if you find a function's address in the GD 2.113 binary, you can use that address in your mod and it will work for anyone with GD 2.113 installed.

> :warning: TODO: Rest of the damn handbook
