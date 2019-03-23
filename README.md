Name
====
lua-laxjson - Lua bindings to [liblaxjson](https://github.com/andrewrk/liblaxjson)
for LuaJIT using FFI.

[liblaxjson](https://github.com/andrewrk/liblaxjson) is a relaxed streaming JSON parser
C library. You don't have to buffer the entire JSON string in memory before parsing it.

Status
======
This library is still experimental and under early development.

Synopsis
========
````lua
local ffi = require "ffi"
local laxjson = require "laxjson"
local C = ffi.C

-- If you don't declare callback functions, default callbacks are used.
-- local laxj = laxjson.new()
local laxj = laxjson.new {
   on_string = function (ctx, ltype, value, length)
      local type_name = ltype == C.LaxJsonTypeProperty and "property" or "string"
      print(type_name..": "..ffi.string(value))
      return 0
   end,
   on_number = function (ctx, x)
      print(x)
      return 0
   end,
   on_primitive = function (ctx, ltype)
      local type_name
      if ltype == C.LaxJsonTypeTrue then
         type_name = "true"
      elseif ltype == C.LaxJsonTypeFalse then
         type_name = "false"
      else
         type_name = "null"
      end
      print("primitive: "..type_name)
      return 0
   end,
   on_begin = function (ctx, ltype)
      local type_name = ltype == C.LaxJsonTypeArray and "array" or "object"
      print("begin "..type_name)
      return 0
   end,
   on_end = function (ctx, ltype)
      local type_name = ltype == C.LaxJsonTypeArray and "array" or "object"
      print("end "..type_name)
      return 0
   end
}

local amt_read
local f = io.open("menu.json", "r")
while true do
    local buf = f:read(32)
    if not buf then break end
    amt_read = #buf
    local ok, err = laxj:feed(amt_read, buf)
    if not ok then
        print(string.format("Line %d, column %d: %s",
                            laxj.line, laxj.column, err))
        laxj:free()
        return
    end
end

local ok, err = laxj:eof()
if not ok then
    print(string.format("Line %d, column %d: %s",
                        laxj.line, laxj.column, err))
end

laxj:free()
````

Installation
============
To install `lua-laxjson` you need to install
[liblaxjson](https://github.com/andrewrk/liblaxjson#installation)
with shared libraries firtst.
Then you can install `lua-laxjson` by placing `laxjson.lua` to
your lua library path.

Methods
=======

new
---
`syntax: laxj = laxjson.new(obj)`

Create laxjson context.

free
----
`syntax: laxj:free()`

Destroy laxjson context.

feed
----
`syntax: ok, err = laxj:feed()`

Feed string to parse.

eof
---
`syntax: ok, err = laxj:eof()`

Check EOF.

Author
======
Soojin Nam jsunam@gmail.com

License
=======
Public Domain
