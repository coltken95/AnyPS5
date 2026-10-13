# libSceNpCppWebApi technical debt

### Unknown function info

- [sce::Np::CppWebApi::Common::IntrusivePtr](../../../core/libs/prx/libSceNpCppWebApi/Export.cpp) (libSceNpCppWebApi) - the 0x18-byte layout (object, deleter, `LibContext*`) and the offset of the 32-bit reference count in the pointee (0x00, 0x30 for `Vector`) come from the module bundled with one title (PPSA21564); other SDK versions are not checked. The destructor throws on the last reference of a pointer without a deleter: destroying the object needs its destructor and the `LibContext` allocator, which are not implemented
- [sce::Np::CppWebApi::Common::String](../../../core/libs/prx/libSceNpCppWebApi/Export.cpp) (libSceNpCppWebApi) - the 0x20-byte layout (32-bit reference count, buffer at 0x08, buffer size at 0x10, `LibContext*` at 0x18) comes from the module bundled with one title (PPSA21564); other SDK versions are not checked. The destructor throws for a non-null buffer: releasing it needs the `LibContext` allocator, which is not implemented. `append` (NID `BgDtc93upnM`), which PPSA03671 imports, is not exported for the same reason

### Functional

- [sce::Np::CppWebApi::Common::initialize](../../../core/libs/prx/libSceNpCppWebApi/Export.cpp) (libSceNpCppWebApi) - succeeds without reading `InitParams` and writes 1 to the first 8 bytes of the `LibContext`, which `LibContext()` and `terminate` set to 0; no memory pool or allocator is created. The `LibContext` and `InitParams` layouts are unknown. Requests and factories return `0x80552901` (`SCE_NP_WEBAPI_ERROR_UNAVAILABLE`) as on a console without a PSN connection; the console's code for that case is not measured
- [libSceNpCppWebApi](../../../core/libs/prx/libSceNpCppWebApi/Export.cpp) is a host replacement for titles that do not ship `libSceNpCppWebApi.prx` in `sce_module/` (PPSA03671); a title that ships it loads its own copy, and the host library is left out of `DT_NEEDED`. It exports the Leaderboards and TitleCloudStorage entry points PPSA21564 imports; of the 147 PPSA03671 imports, 129 are not exported
