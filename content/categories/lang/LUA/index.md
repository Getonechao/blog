+++
title= "LUA：C/C++的脚本语言"
description= "文章简介"
date= 2023-04-24T14:58:29+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= ["lang"]

tags=  [" "," "]

+++

# 一、c/c++中调用LUA

## 1.开启控制台

~~~
   // 打开lua
    LUA=luaL_newstate();
    // 导入LUA标准库
    luaL_openlibs(LUA);
    // 打开脚本文件
    if(luaL_dofile(LUA, "LidarConfig.lua")!=LUA_OK)
    {
        std::cerr<<"sorry lua file load and run failed"<<std::endl;
        exit(-1);
    }

~~~



## 2.入栈操作

~~~
void lua_pushnil      (lua_State *L);
void lua_pushboolean  (lua_State *L, int bool);
void lua_pushnumber   (lua_State *L, lua_Number n);
void lua_pushinteger  (lua_State *L, lua_Integer n);
void lua_pushunsigned (lua_State *L, lua_Unsigned n);
void lua_pushlstring  (lua_State *L, const char *s, size_t len);
void lua_pushstring   (lua_State *L, const char *s);
~~~

## 2.查询栈里面的元素

~~~
int lua_is* (lua_State * L, int index);

LUA_API int             (lua_isnumber) (lua_State *L, int idx);
LUA_API int             (lua_isstring) (lua_State *L, int idx);
LUA_API int             (lua_iscfunction) (lua_State *L, int idx);
LUA_API int             (lua_isinteger) (lua_State *L, int idx);
LUA_API int             (lua_isuserdata) (lua_State *L, int idx);
LUA_API int             (lua_type) (lua_State *L, int idx);
LUA_API const char     *(lua_typename) (lua_State *L, int tp);
~~~

## 3.取值操作

~~~
LUA_API lua_Number      (lua_tonumberx) (lua_State *L, int idx, int *isnum);
LUA_API lua_Integer     (lua_tointegerx) (lua_State *L, int idx, int *isnum);
LUA_API int             (lua_toboolean) (lua_State *L, int idx);
LUA_API const char     *(lua_tolstring) (lua_State *L, int idx, size_t *len);
LUA_API lua_Unsigned    (lua_rawlen) (lua_State *L, int idx);
LUA_API lua_CFunction   (lua_tocfunction) (lua_State *L, int idx);
LUA_API void	       *(lua_touserdata) (lua_State *L, int idx);
LUA_API lua_State      *(lua_tothread) (lua_State *L, int idx);
LUA_API const void     *(lua_topointer) (lua_State *L, int idx);
~~~

## 4.LUA脚本入栈操作

~~~
LUA_API int (lua_getglobal) (lua_State *L, const char *name);
LUA_API int (lua_gettable) (lua_State *L, int idx);
LUA_API int (lua_getfield) (lua_State *L, int idx, const char *k);
LUA_API int (lua_geti) (lua_State *L, int idx, lua_Integer n);

ps
me = { name = "zilongshanren", age = 27}

lua_getglobal(L,table);
lua_getfield(L, -1, "age");//-1是table的索引

等同于

lua_getglobal(L,table);
//压入另一个key:age
lua_pushstring(L, "age");
//取出-2位置的table,把table[age]的值压入栈
lua_gettable(L, -2)
~~~

