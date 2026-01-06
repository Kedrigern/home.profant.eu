# uv benchmarks

![](https://private-user-images.githubusercontent.com/1309177/359563195-84118aaa-d030-4e29-8f1e-9483091ceca3.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTI0ODMxMjYsIm5iZiI6MTc1MjQ4MjgyNiwicGF0aCI6Ii8xMzA5MTc3LzM1OTU2MzE5NS04NDExOGFhYS1kMDMwLTRlMjktOGYxZS05NDgzMDkxY2VjYTMucG5nP1gtQW16LUFsZ29yaXRobT1BV1M0LUhNQUMtU0hBMjU2JlgtQW16LUNyZWRlbnRpYWw9QUtJQVZDT0RZTFNBNTNQUUs0WkElMkYyMDI1MDcxNCUyRnVzLWVhc3QtMSUyRnMzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA3MTRUMDg0NzA2WiZYLUFtei1FeHBpcmVzPTMwMCZYLUFtei1TaWduYXR1cmU9ZjkxZjMyMGYwMjYyNjk0ZDU1OTQ1NmUzNmU0ZjY1NjMwOGM5MWU5NmZhNGYzYTYwYWY0YmQ1YWY5OTYyMGZhMiZYLUFtei1TaWduZWRIZWFkZXJzPWhvc3QifQ.JpEULTdShy2pQCDrUMODl8x9ocMmEkeqD4vVHHD6Yec)

-- Source [uv vs poetry](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md)

## uv vs pyenv

- PyEnv install python version system wide.
- uv install localy (under the user).

| Tool  | Executed in | CPU time |
|-------|------------:|---------:|
| uv    |        3.11 |     1.20 |
| pyenv |       94.40 |   343.14 |

In both cases, the values represent the average of two measurements, in seconds.

!!! Benchmark
    First, I attempted to install Python 3.8. This version had never been present on my system before.

    === "uv"

        ```console
        $ time uvx --python 3.8 python -c 'print("Hello world")'
        ________________________________________________________
        Executed in    3.34 secs    fish           external
        usr time    1.19 secs    0.68 millis    1.19 secs
        sys time    0.54 secs    1.38 millis    0.54 secs

        $ uv python uninstall 3.8
        Searching for Python versions matching: Python 3.8
        Uninstalled Python 3.8.20 in 119ms
        - cpython-3.8.20-linux-x86_64-gnu

        $ time uvx --python 3.8 python -c 'print("Hello world")'
        Hello world

        ________________________________________________________
        Executed in    2.88 secs    fish           external
        usr time    1.20 secs    0.02 millis    1.20 secs
        sys time    0.51 secs    2.06 millis    0.50 secs
        ```
        
    === "PyEnv"

        ```python
        def method_b():
            return "complex but powerful"
        ```
        
        **Výhody:**
        - Výkonné
        - Flexibilní
