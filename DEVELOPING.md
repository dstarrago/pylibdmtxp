## Development

```
python -m venv venv
source ./venv/bin/activate
pip install -U pip
pip install -r requirements.txt

python -m pytest --verbose --cov=pylibdmtxp --cov-report=term-missing --cov-report=html pylibdmtxp
python -m pylibdmtxp.scripts.read_datamatrix pylibdmtxp/tests/datamatrix.png
```

### Test matrix of supported Python versions

Run tox

```
rm -rf .tox && tox
```

### Windows

Save the 32-bit and 64-bit `libdmtx.dll` files to `libdmtx-32.dll` and
`libdmtx-64.dll` respectively, in the `pylibdmtxp` directory.
The `load_pylibdmtxp` function in `wrapper.py` looks for the appropriate `DLL`s.
The appropriate `DLL` is packaged up into the wheel build and is installed
alongside the package source. This strategy allows the same method to be used
when `pylibdmtxp` is run from source, as an installed package and when included
in a frozen binary.

## Releasing

1. Build

    Create source and wheel builds. The `win32` and `win_amd64` wheels will
    contain the appropriate `libdmtx.dll`.

    ```
    rm -rf build dist MANIFEST.in pylibdmtxp.egg-info
    cp MANIFEST.in.all MANIFEST.in
    ./setup.py bdist_wheel

    cat MANIFEST.in.all MANIFEST.in.win32 > MANIFEST.in
    ./setup.py bdist_wheel --plat-name=win32

    # Remove these dirs to prevent win32 DLL from being included in win64 build
    rm -rf build pylibdmtxp.egg-info
    cat MANIFEST.in.all MANIFEST.in.win64 > MANIFEST.in
    ./setup.py bdist_wheel --plat-name=win_amd64

    rm -rf build MANIFEST.in pylibdmtxp.egg-info
    ```

2. Release to pypitest (see https://packaging.python.org/guides/using-testpypi/)

    ```
    twine upload -r testpypi dist/*
    ```

3. Test the release to pypitest

    * Check https://testpypi.python.org/pypi/pylibdmtxp/

    * If you are on Windows

    ```
    c:\python27\python.exe -m venv test1
    test1\scripts\activate
    ```

    * Install dependencies that are not on testpypi.python.org.
    If you are on Python 2.x, these are mandatory

    ```
    pip install enum34 pathlib
    ```

    * Pillow for tests and `read_datamatrix`. We can't use the
    `pip install pylibdmtxp[scripts]` form here because `Pillow` will not be
    on testpypi.python.org

    ```
    pip install Pillow
    ```

    * Install the package itself

    ```
    pip install --index https://testpypi.python.org/simple pylibdmtxp
    ```

    * Test

    ```
    read_datamatrix --help
    read_datamatrix <path-to-datamatrix.png>
    ```

4. If all is well, release to PyPI

    ```
    twine upload dist/*
    ```

    * Check https://pypi.python.org/pypi/pylibdmtxp/

    * Install!

    ```
    pip install pylibdmtxp[scripts]
    ```
