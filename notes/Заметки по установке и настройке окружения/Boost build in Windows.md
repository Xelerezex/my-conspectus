1. Run Windows PowerShell
2. cd downloaded boost directory
3. Run command: ./bootstrap
4. Open file: project-config.jam
5. Set line: using msvc : 14.3 ; (for Visual Studio 2022)
6. Save file project-config.jam
7. Run command: ./b2 release debug threading=multi link=static --build-type=complete --toolset=msvc-14.3 address-model=64 stage install --prefix=C:\workspace\libs\boost_1_83_0_msvc_14_3
8. Run command (if needed Python): ./b2 release debug threading=multi --with-python --python=3.8 link=shared --build-type=complete --toolset=msvc-14.3 address-model=64 stage install --prefix=C:\workspace\libs\boost_1_83_0_msvc_14_3