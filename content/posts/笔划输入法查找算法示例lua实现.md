---
title: "笔划输入法查找算法示例（Lua实现）"
date: 2012-08-04T17:58:00+08:00
draft: false
---

公司同事最近利用谷歌拼音输入法源代码实现了自己的拼音输入法，经过了解，最核心的就是一个trie（词典树）的构造和检索（这里不太介绍trie树了，google一搜一大把），于是今天就想实现了一个笔划输入法。大概的想法是：


1. 找一个所有汉字或者一、二级汉字的笔顺数据库
2. 用Lua将数据库读进来，构造一个trie树
	1. 每一个节点存一个笔划
	2. 每一个节点带一个子节点集合
	3. 每一个节点带一个汉字集合，表示到这一级时所有笔划组成的完整汉字
3. 检索时根据用户输入的笔划，检索到一个节点，然后按笔划顺序遍历子树
	1. 遍历子树可以给出所有以这些笔划开始的所有汉字，但是总不能一下显示出来吧，所以需要一个迭代器，每调用一次给出一个可能的值，这个迭代器用C实 现比较复杂，但是用Lua实现简直就是小意思，直接将遍历子树的函数封装到一个coroutine中，每找到一个汉字就 yield(汉字) 即可


### 笔顺数据库


CSDN上可以下载到 <http://download.csdn.net/detail/yyjlan/3766691>


下载的mdb格式，我不太喜欢，Lua也不太喜欢。由于luasql支持odbc，所以可以将mdb文件加入到odbc数据源，然后载入后转成sqlite3的格式，方便以后使用，转换代码如下




```
 1 require "luasql.odbc"
 2 require "luasql.sqlite3"
 3 
 4 odbc_env = luasql.odbc()
 5 
 6 -- 将Access文件在控制面板->管理工具->数据源 中增加到用户DSN，名称是hzbs
 7 odbc_conn = odbc_env:connect("hzbs")
 8 odbc_cur = odbc_conn:execute("SELECT \* FROM hzbs;")
 9 
10 sqlite_env = luasql.sqlite3()
11 sqlite_conn = sqlite_env:connect("hzbs.sqlite3.db")
12 sqlite_conn:execute("CREATE TABLE hzbs (id INTEGER primary key, hanzi TEXT, stroke\_number INTEGER, stroke\_order TEXT, unicode TEXT, gbk TEXT);")
13 sqlite_conn:setautocommit(false) -- start transaction
14 
15 record = {}
16 while odbc_cur:fetch(record, "n") do
17     local id = record[1]
18     local hanzi = record[2]
19     local stroke_number = record[3]
20     local stroke_order = record[4]
21     local unicode = record[5]
22     local gbk = record[6]
23     sqlite_conn:execute("INSERT INTO hzbs(id, hanzi, stroke\_number, stroke\_order, unicode, gbk) VALUES(" .. id .. ",\'" .. hanzi .. "\'," .. stroke_number .. ",\'" .. stroke_order .. "\',\'" .. unicode .. "\',\'" .. gbk .. "\');")
24 end
25 
26 sqlite_conn:commit() -- commit the transaction
27 sqlite\_conn:close()
28 
29 odbc\_cur:close()
30 odbc\_conn:close()
31 odbc_env:close()
```


### 构造子树与检索


多的不说，直接看代码吧。代码写得有点乱，不过凑合看是没什么问题的。要运行代码必须要先安装 LuaForWindows




```
 1 require "luasql.sqlite3"
 2 require "wx"
 3 
 4 
 5 function \_T(s)
 6     return s
 7 end
 8 
 9 -- enum stroke\_t {
 10 local stroke_root = 0 -- for trie root, not a valid stroke
 11 local stroke_heng = 1
 12 local stroke_shu = 2
 13 local stroke_pie = 3
 14 local stroke_na = 4
 15 local stroke_zhe = 5
 16 local stroke_max = 5
 17 local stroke_text = {\_T"一", \_T"丨", \_T"丿", \_T"丶", \_T"乛"}
 18 -- }
 19 
 20 function new\_node(stroke)
 21     return {stroke=stroke,  -- see stroke definition
 22         subnodes = {},  -- next strokes
 23         hanzis={} -- two or more hanzi could have the same stroke order
 24  }
 25 end
 26 
 27 function new\_trie()
 28     return new\_node(stroke\_root)
 29 end
 30 
 31 -- insert hanzi and create the trie
 32 function insert\_hanzi(node, stroke\_order, hanzi)
 33     local stroke, not\_found\_index
 34     for i = 1, #stroke_order do
 35         stroke = tonumber(stroke\_order:sub(i,i))
 36         if node.subnodes[stroke] then
 37             node = node.subnodes[stroke]
 38         else
 39             not_found_index = i
 40             break
 41         end
 42     end
 43     if not_found_index then
 44         for i = not_found_index, #stroke_order do
 45             stroke = tonumber(stroke\_order:sub(i,i))
 46             node.subnodes[stroke] = new\_node(stroke)
 47             node = node.subnodes[stroke]
 48         end
 49     end
 50     table.insert(node.hanzis, hanzi)
 51 end
 52 
 53 -- 看看strokes数组组成的笔划顺序的节点是否存在，如果存在则返回节点
 54 function find\_node(root, strokes)
 55     local node = root
 56 
 57     if #strokes < 1 then
 58         return nil
 59     end
 60 
 61     for i, stroke in ipairs(strokes) do
 62         if node.subnodes[stroke] then
 63             node = node.subnodes[stroke]
 64         else
 65             return nil
 66         end
 67     end
 68     return node
 69 end
 70 
 71 function db\_to\_trie(db\_name)
 72     local env = luasql.sqlite3()
 73     local conn = env:connect(db\_name)
 74     local cur = conn:execute("SELECT hanzi,stroke\_order FROM hzbs;")
 75     local trie = new\_trie()
 76 
 77     record = {}
 78     while cur:fetch(record, "a") do
 79  insert\_hanzi(trie, record.stroke\_order, record.hanzi)
 80     end
 81 
 82  cur:close()
 83  conn:close()
 84  env:close()
 85 
 86     return trie
 87 end
 88 
 89 function get\_hanzi\_enumerator(root)
 90     local traverse
 91 
 92     traverse = function(node)
 93         for i = 1, #node.hanzis do
 94             coroutine.yield(node.hanzis[i])
 95         end
 96 
 97         for stroke = 1, stroke_max do
 98             if node.subnodes[stroke] then
 99  traverse(node.subnodes[stroke])
100             end
101         end
102     end
103     local co = coroutine.create(function () traverse(root) end)
104 
105     return (function ()
106         local ret, hanzi = coroutine.resume(co)
107         if not ret then -- already stopped
108             return nil
109         elseif hanzi == nil then -- the last call, no yield and no return value
110             return nil
111         else
112             return hanzi
113         end
114     end)
115 end
116 
117 ---------------------------------------------------------------
118 -- GUI
119 local new_id = (function ()
120     local id = wx.wxID\_HIGHEST
121     return (function ()
122         id = id + 1
123         return id
124     end)
125 end)()
126 
127 dialog = wx.wxDialog(wx.NULL, new_id(), \_T"Lua笔划输入法演示",
128  wx.wxDefaultPosition, wx.wxDefaultSize)
129 panel = wx.wxPanel(dialog, wx.wxID\_ANY)
130 local main_sizer = wx.wxBoxSizer(wx.wxVERTICAL)
131 
132 -- 横竖撇捺折 按钮
133 local stroke_label = wx.wxStaticText(panel, new_id(), \_T"可选笔划")
134 local heng_button = wx.wxButton(panel, stroke\_heng, stroke\_text[stroke\_heng])
135 local shu_button = wx.wxButton(panel, stroke\_shu, stroke\_text[stroke\_shu])
136 local pie_button = wx.wxButton(panel, stroke\_pie, stroke\_text[stroke\_pie])
137 local na_button = wx.wxButton(panel, stroke\_na, stroke\_text[stroke\_na])
138 local zhe_button = wx.wxButton(panel, stroke\_zhe, stroke\_text[stroke\_zhe])
139 
140 local button_sizer = wx.wxBoxSizer(wx.wxHORIZONTAL)
141 button_sizer:Add(stroke_label, 0, wx.wxALIGN_LEFT+wx.wxALL, 5)
142 button_sizer:Add(heng_button, 0, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
143 button_sizer:Add(shu_button, 0, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
144 button_sizer:Add(pie_button, 0, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
145 button_sizer:Add(na_button, 0, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
146 button_sizer:Add(zhe_button, 0, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
147 
148 main_sizer:Add(button_sizer, 0, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
149 
150 -- 输入笔划列表
151 local input_label = wx.wxStaticText(panel, new_id(), \_T"输入笔划")
152 local input_textctrl = wx.wxTextCtrl(panel, new_id(), "",
153  wx.wxDefaultPosition, wx.wxDefaultSize, wx.wxTE\_READONLY)
154 local input_backspace_button = wx.wxButton(panel, new_id(), \_T"退格")
155 local input_clear_button = wx.wxButton(panel, wx.wxID_CANCEL, \_T"清除")
156 
157 local input_sizer = wx.wxBoxSizer(wx.wxHORIZONTAL)
158 input_sizer:Add(input_label, 0, wx.wxALIGN_LEFT+wx.wxALL, 5)
159 input_sizer:Add(input_textctrl, 1, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
160 input_sizer:Add(input_backspace_button, 0, wx.wxALL, 5)
161 input_sizer:Add(input_clear_button, 0, wx.wxALL, 5)
162 main_sizer:Add(input_sizer, 1, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
163 
164 -- 备选汉字
165 local candidate_label = wx.wxStaticText(panel, new_id(), \_T"备选汉字")
166 local candidate_sizer = wx.wxBoxSizer(wx.wxHORIZONTAL)
167 candidate_sizer:Add(candidate_label, 0, wx.wxALIGN_LEFT+wx.wxALL, 5)
168 
169 local candidate_number = 5
170 function create\_candidate\_btn(num)
171     local textctrls = {}
172     for i= 1, num do
173         textctrls[i] = wx.wxButton(panel, new_id(), "")
174         candidate_sizer:Add(textctrls[i], 1, wx.wxALIGN_LEFT+wx.wxALL+wx.wxEXPAND, 5)
175     end
176     textctrls.start_id = textctrls[1]:GetId()
177     textctrls.end_id = textctrls.start_id + candidate_number - 1
178     return textctrls
179 end
180 local candidate_textctrls = create\_candidate\_btn(candidate\_number)
181 main_sizer:Add(candidate_sizer, 1, wx.wxALIGN_LEFT+wx.wxALL+wx.wxEXPAND, 5)
182 
183 -- 选择输出的汉字
184 local output_textctrl = wx.wxTextCtrl(panel, new_id(), "", wx.wxDefaultPosition,
185     wx.wxSize(0, 100), wx.wxTE\_MULTILINE)
186 local output_sizer = wx.wxBoxSizer(wx.wxHORIZONTAL)
187 output_sizer:Add(output_textctrl, 1, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 5)
188 main_sizer:Add(output_sizer, 0, wx.wxALIGN_LEFT+wx.wxEXPAND+wx.wxALL, 0)
189 
190 main\_sizer:SetSizeHints(dialog)
191 dialog:SetSizer(main\_sizer)
192 
193 -- 必须加，否则不能正确退出程序
194 dialog:Connect(wx.wxEVT\_CLOSE\_WINDOW,
195     function (event)
196  dialog:Destroy()
197  event:Skip()
198     end)
199 
200 -- 读入笔划数据库
201 local trie = db_to_trie("hzbs.sqlite3.db")
202 
203 -- 输入的stroke数组
204 input_strokes = {}
205 get_next_candidate = nil
206 
207 function update\_candidate()
208     if get_next_candidate == nil then
209         for _,textctrl in ipairs(candidate_textctrls) do
210             textctrl:SetLabel("")
211         end
212     else
213         for _,textctrl in ipairs(candidate_textctrls) do
214             local hanzi = get\_next\_candidate()
215             if hanzi then
216  textctrl:SetLabel(hanzi)
217             else
218                 textctrl:SetLabel("")
219             end
220         end
221     end
222 end
223 
224 function update\_input()
225     local text = {}
226     for _,stroke in ipairs(input_strokes) do
227         table.insert(text, stroke\_text[stroke])
228     end
229 
230     input_textctrl:SetValue(table.concat(text, " "))
231 end
232 
233 function insert\_stroke(stroke)
234     table.insert(input\_strokes, stroke);
235     local node = find\_node(trie, input\_strokes)
236     if node == nil then
237         table.remove(input_strokes) -- 删除不合法的输入
238         -- BEEP
239     else
240         get_next_candidate = get\_hanzi\_enumerator(node)
241  update\_input()
242  update\_candidate()
243     end
244 end
245 
246 function remove\_stroke()
247     table.remove(input\_strokes)
248     local node = find\_node(trie, input\_strokes)
249     if node == nil then
250         get_next_candidate = nil
251     else
252         get_next_candidate = get\_hanzi\_enumerator(node)
253     end
254 
255  update\_input()
256  update\_candidate()
257 end
258 
259 function clear\_stroke()
260     input_strokes = {}
261     get_next_candidate = nil
262  update\_input()
263  update\_candidate()
264 end
265 
266 dialog:Connect(wx.wxID\_ANY, wx.wxEVT\_COMMAND\_BUTTON\_CLICKED,
267     function(event)
268         local id = event:GetId()
269         if id <= stroke_max then
270  insert\_stroke(id)
271         elseif id >= candidate_textctrls.start_id and id <= candidate_textctrls.end_id then
272             output_textctrl:AppendText(candidate_textctrls[id-candidate_textctrls.start_id+1]:GetLabel())
273  clear\_stroke()
274         elseif id == input_backspace_button:GetId() then
275  remove\_stroke()
276         elseif id == input_clear_button:GetId() then
277  clear\_stroke()
278         end
279     end)
280 
281 dialog:Connect(wx.wxID_ANY, wx.wxEVT_KEY_DOWN, function (event)
282     local key = event:GetKeyCode()
283     local callbacks = { }
284     callbacks[wx.WXK_NUMPAD7] = function ()
285  insert\_stroke(stroke\_heng)
286     end
287     callbacks[wx.WXK_NUMPAD8] = function ()
288  insert\_stroke(stroke\_shu)
289     end
290     callbacks[wx.WXK_NUMPAD9] = function ()
291  insert\_stroke(stroke\_pie)
292     end
293     callbacks[wx.WXK_NUMPAD4] = function ()
294  insert\_stroke(stroke\_na)
295     end
296     callbacks[wx.WXK_NUMPAD5] = function ()
297  insert\_stroke(stroke\_zhe)
298     end
299     callbacks[wx.WXK_BACK] = function ()
300  remove\_stroke()
301     end
302     for i = 1, candidate_number do
303         callbacks[i - 1 + string.byte("1")] = function ()
304  output\_textctrl:AppendText(candidate\_textctrls[i]:GetLabel())
305  clear\_stroke()
306         end
307     end
308 
309     if callbacks[key] then
310  callbacks[key]()
311     end
312 end)
313 
314 -- wxwindgets比较特殊，子窗口的按键是发不到主窗口的，需要这样处理下
315 function process\_children\_keydown\_event(parent, processer)
316     local wnd
317     local wlist = parent:GetChildren()
318 
319     for i = 0, wlist:GetCount()-1 do
320         wnd = wlist:Item(i):GetData():DynamicCast("wxWindow")
321  wnd:SetNextHandler(processer)
322  process\_children\_keydown\_event(wnd, processer)
323     end
324 end
325 
326 process\_children\_keydown\_event(dialog, dialog)
327 
328 
329 dialog:Centre()
330 dialog:Show(true)
331 input_textctrl:SetFocus() --放这里没有响声
332 
333 wx.wxGetApp():MainLoop()
```


 


### 打包下载


源代码包和sqlite3数据库可以在[这里](http://files.cnblogs.com/windtail/ime-win32.rar)下载


