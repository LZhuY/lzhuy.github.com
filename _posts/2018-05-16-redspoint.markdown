---
layout: post
title:  "基于B+树实现的小红点"
date:   2018-05-16 22:21:49
categories: data_structure
tags: data_structure
---

基于B+树实现的小红点，觉得是个不错的思路，具体代码如下。

{% highlight ruby %}

local string = string
local sfind = string.find
local ssub  = string.sub
local slen  = string.len
local sgsub = string.gsub
local smatch = string.match

local M = {}
local N = {}

function string.trim(s)
    return smatch(s, "^%s*(.-)%s*$")
end

local trim = string.trim
function string.split(szFullString, szSeparator, needtrim, func)
    local nFindStartIndex = 1
    local nSplitIndex = 1
    local nSplitArray = {}
    while true do
        local nFindLastIndex = sfind(szFullString, szSeparator, nFindStartIndex)
        if not nFindLastIndex then
            local str = ssub(szFullString, nFindStartIndex, slen(szFullString))
            if needtrim then str=trim(str) end
            nSplitArray[nSplitIndex] = func and func(str) or str
            break
        end

        local str = ssub(szFullString, nFindStartIndex, nFindLastIndex - 1)
        if needtrim then str=trim(str) end

        nSplitArray[nSplitIndex] = func and func(str) or str
        nFindStartIndex = nFindLastIndex + slen(szSeparator)
        nSplitIndex = nSplitIndex + 1
    end
    return nSplitArray
end

function N:new(key, parent)
	local o = {key=key, parent=parent, childs={}, count=0}
	setmetatable(o, {__index = self})
	return o
end

function N:waker(path, result)
	for key, child in pairs(self.childs) do 
		local tpath = path .. "/" .. key
		if not next(child.childs) then
			table.insert(result, tpath)
		else
			child:waker(tpath, result)
		end
	end
end

function N:rever()
end

function Node(key, parent)
	return N:new(key, parent)
end
function M:new()
	local o = {root=Node("root", nil)}
	setmetatable(o, {__index = self})
	return o
end

function M:mark(path, cnt)
	local tmp = self.root
	tmp.count = tmp.count+cnt
	for _, v in ipairs( string.split(path, "/") ) do
		tmp.childs[v] = tmp.childs[v] or Node(v, tmp)
		tmp = tmp.childs[v]
		tmp.count = tmp.count+cnt
	end
end

function M:unmark(path, cnt)
	local tmp = self.root
	--tmp.count = tmp.count - cnt
	for _, v in ipairs(string.split(path, "/")) do 
		if not tmp.childs[v] then
			return 0 
		end
		tmp = tmp.childs[v]
		tmp.count = tmp.count - cnt
		if tmp.count <= 0 then 
			tmp.parent.childs[tmp.key] = nil
		end
	end
end

function M:getAllDate()
	local result = {}
	self.root:waker("", result)
	return result
end

tree = M:new()
tree:mark("a/b/c", 10)
tree:mark("a/b/d", 10)
tree:mark("a/b/e", 10)
tree:unmark("a/b/c", 10)

result = tree:getAllDate()
for k,v in pairs(result) do 
	print(k,v)
end

{% endhighlight %}