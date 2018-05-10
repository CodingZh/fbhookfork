# fbhookfork
从 fb 的 profilo 项目里提取出来的plt hook 库，自己用

该库不支持arm 64的库,比如无法支持 `/system/lib64/libc.so` 里的方法

# Use

```cpp

//需要 hook 的方法
std::vector<std::pair<char const *, void *>> &getFunctionHooks() {
    static std::vector<std::pair<char const *, void *>> functionHooks = {
            {"write",       reinterpret_cast<void *>(&write_hook)},
            {"__write_chk", reinterpret_cast<void *>(__write_chk_hook)},
    };
    return functionHooks;
}

//这里传入 hook 方法的库地址和本身库的地址
std::unordered_set<std::string> &getSeenLibs() {
    static bool init = false;
    static std::unordered_set<std::string> seenLibs;

    if (!init) {
        seenLibs.insert("/system/lib/libc.so");

        Dl_info info;
        if (!dladdr((void *) &getSeenLibs, &info)) {
            ALOG("Failed to find module name");
        }
        if (info.dli_fname == nullptr) {
            throw std::runtime_error("could not resolve current library");
        }

        seenLibs.insert(info.dli_fname);
        init = true;
    }
    return seenLibs;
}

//调用hookLoadedLibs即可
void hookLoadedLibs() {
    auto &functionHooks = getFunctionHooks();
    auto &seenLibs = getSeenLibs();
    facebook::profilo::hooks::hookLoadedLibs(functionHooks, seenLibs);
}

```
