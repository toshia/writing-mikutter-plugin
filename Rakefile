# coding: utf-8
require 'yaml'
require 'erb'
require 'ostruct'
task :spell do
  spells = YAML.load_file(File.join(__dir__, '_posts/spell.yml'))
  open(File.join(__dir__, '_posts/2017-11-28-spell.markdown'), 'w') do |out|
    out << <<EOF
---
layout: post
title:  "Spellリファレンス"
date:   2017-11-28 00:00:00 +0900
categories:
- reference
---
EOF
    spells.each do |spell|
      erb = ERB.new(<<"EOF")
## <a id="<%= [name, *target.map{|x|x['name']}].join('-') %>"></a><%= name %>[<%= target.map{|x|x['name']}.join(',') %>]

### 説明

<%= description %>

### 対象

<% target.each do |opt| %>
- **<%= opt['name'] %>** <%= opt['description'] %>
<% end %>

<% if option and !option.empty? %>
### オプション

<% option.each do |opt| %>
- **<%= opt['name'] %>** <%= opt['description'] %>
<% end %>

<% end %>
### 戻り値

<%= returns %>

### 例

<% if example.size == 1 %>
```ruby
<%= example.first['code'] %>
```
結果
```
<%= example.first['result'] %>
```
<% else %>
<% example.each do |ex| %>
#### <%= ex['title'] %>
```ruby
<%= ex['code'] %>
```
結果
```
<%= ex['result'] %>
```

<% end # example.each %>
<% end # if %>

EOF
      out << erb.result(OpenStruct.new(spell).instance_eval{binding})
    end
  end
end
