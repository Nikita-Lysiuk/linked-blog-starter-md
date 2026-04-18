1) Skrypt napisany na python:
``` python
#!/usr/bin/env python3
import os
import sys

def create_structure(root_path, folders):
	try:
        if not os.path.exists(root_path):
            os.makedirs(root_path)
        for folder in folders:
            path = os.path.join(root_path, folder)
            os.makedirs(path, exist_ok=True)
        print("\nSuccessfull!")
    except Exception as e:
        print(f"Error: {e}")
if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Usage: python mksdir.py <root> <folder1> <folder2> ...")
        print("Example: python mksdir.py MyProject src tests docs")
        sys.exit(1)
    root = sys.argv[1]
    subfolders = sys.argv[2:]
    create_structure(root, subfolders)
```
2) yaml configuracji snapcraft
``` yaml
name: mksdir-tool
base: core22
version: '1.0'
summary: Simple tool for creating folder structures
description: CLI utility which create any folder structure.
title: Mksdir Tool

grade: stable
confinement: strict

apps:
  mksdir-tool:
    command: bin/mksdir-tool
    plugs:
      - home
      - network

parts:
  my-script:
    plugin: dump
    source: .
    stage-packages:
      - python3-minimal
    override-build: |
      mkdir -p $SNAPCRAFT_PART_INSTALL/bin
      cp mksdir.py $SNAPCRAFT_PART_INSTALL/bin/mksdir-tool
      chmod +x $SNAPCRAFT_PART_INSTALL/bin/mksdir-tool

```
3) Stworzenie pakietu snap
![[{91D01A31-7FCE-4466-A64E-6CFAE33E0570}.png]]
4) Instalacja pakietu i uruchomienie skryptu
![[{15214C43-FE71-4491-A92F-7B0536239A16}.png]]