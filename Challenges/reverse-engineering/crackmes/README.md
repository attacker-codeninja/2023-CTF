# crackmes

## crackme01

Can you think of a way to determine the length of the correct password and use that information to recover the correct password?

**Hint**: The program uses the `strncmp()` function to compare the user input password with the correct password stored in memory. `strncmp()` compares strings up to a certain length specified as its third argument.

<details>
  <summary>Expant to view the solution</summary>

  ---

  You can solve the challenge using `strings` and `ltrace`:

## #1 Using strings

1. Open a terminal window and navigate to the directory containing the `crackme01` executable.

2. Use the strings command to search for any printable `strings` in the executable, as follows:

```shell
strings crackme01
```

This should display a list of all printable strings including the correct password (in this case, `ctf2023`).

## #2 Using ltrace

1. Run the following command to use `ltrace` to trace the program's library calls while passing an arbitrary string as the argument:

    ```shell
    ltrace ./crackme01 arbitrary_string
    ```

2. The output should show all the library calls made by the program, including the `strncmp()` call that compares the user input with the correct password. Look for a line that looks similar to this:

    ```c
    strncmp("arbitrary_string", "ctf2023", 7) = ...
    ```

    This line shows that `strncmp()` was called with the arbitrary string `("arbitrary_string")` as the first argument and the correct password `("ctf2023")` as the second argument.

3. Note the length of the correct password (in this case, 7), which is passed as the third argument to `strncmp()`.

4. Next, run the `ltrace` command again, this time passing a string of the same length as the correct password (in this case, 7) as the argument. For example:

    ```shell
    ltrace ./crackme01 aaaaaaaa
    ```

    This will cause the program to compare the input string `("aaaaaaaa")` with the correct password ("ctf2023") using `strncmp()`.

5. The output should again show the `strncmp()` call, but this time with different arguments. Look for a line that looks similar to this:

```c
strncmp("aaaaaaaa", "ctf2023", 7) = ...
```

This line shows that `strncmp()` was called with the input string `("aaaaaaaa")` as the first argument and the correct password `("ctf2023")` as the second argument.

</details>  
