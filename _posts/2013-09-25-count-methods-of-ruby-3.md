---
layout: post
title: "Rubyでオブジェクトの特徴的なメソッドを知りたい"
description: ""
category: 
tags: 
date: 2013-09-25
published: true
---
{% include JB/setup %}

オブジェクトがどんな挙動をするのか、つまりそのオブジェクトの特徴的なメソッド群をさっと知りたいとします。そんなとき自分は`Object#methods`を呼びます。

{% highlight ruby %}
irb> [1,2,3].methods
=> [:inspect, :to_s, :to_a, :to_ary, :frozen?, :==, :eql?, :hash, :[], :[]=, :at, :fetch, :first, :last, :concat, :<<, :push, :pop, :shift, :unshift, :insert, :each, :each_index, :reverse_each, :length, :size, :empty?, :find_index, :index, :rindex, :join, :reverse, :reverse!, :rotate, :rotate!, :sort, :sort!, :sort_by!, :collect, :collect!, :map, :map!, :select, :select!, :keep_if, :values_at, :delete, :delete_at, :delete_if, :reject, :reject!, :zip, :transpose, :replace, :clear, :fill, :include?, :<=>, :slice, :slice!, :assoc, :rassoc, :+, :*, :-, :&, :|, :uniq, :uniq!, :compact, :compact!, :flatten, :flatten!, :count, :shuffle!, :shuffle, :sample, :cycle, :permutation, :combination, :repeated_permutation, :repeated_combination, :product, :take, :take_while, :drop, :drop_while, :bsearch, :pack, :entries, :sort_by, :grep, :find, :detect, :find_all, :flat_map, :collect_concat, :inject, :reduce, :partition, :group_by, :all?, :any?, :one?, :none?, :min, :max, :minmax, :min_by, :max_by, :minmax_by, :member?, :each_with_index, :each_entry, :each_slice, :each_cons, :each_with_object, :chunk, :slice_before, :lazy, :nil?, :===, :=~, :!~, :class, :singleton_class, :clone, :dup, :taint, :tainted?, :untaint, :untrust, :untrusted?, :trust, :freeze, :methods, :singleton_methods, :protected_methods, :private_methods, :public_methods, :instance_variables, :instance_variable_get, :instance_variable_set, :instance_variable_defined?, :remove_instance_variable, :instance_of?, :kind_of?, :is_a?, :tap, :send, :public_send, :respond_to?, :extend, :display, :method, :public_method, :define_singleton_method, :object_id, :to_enum, :enum_for, :equal?, :!, :!=, :instance_eval, :instance_exec, :__send__, :__id__]
{% endhighlight %}

しかし、この結果にはそのオブジェクトで利用できるすべてのメソッドが含まれているので、これを眺めてそのオブジェクトの特性を推測するのは困難です。

`methods`メソッドの引数に`false`を渡すと、そのオブジェクト固有のメソッドだけが抽出されます。

{% highlight ruby %}
irb> [1,2,3].methods(false)
=> []
{% endhighlight %}

シングルトンメソッドが定義されていない限り、通常何も出力されないので、あまり役に立ちません。

そこで、そのオブジェクトのクラスを調べ、そのクラスに定義されているメソッドを調べることになります。

{% highlight ruby %}
irb> [1,2,3].class
=> Array
irb> Array.instance_methods(false)
=> [:inspect, :to_s, :to_a, :to_ary, :frozen?, :==, :eql?, :hash, :[], :[]=, :at, :fetch, :first, :last, :concat, :<<, :push, :pop, :shift, :unshift, :insert, :each, :each_index, :reverse_each, :length, :size, :empty?, :find_index, :index, :rindex, :join, :reverse, :reverse!, :rotate, :rotate!, :sort, :sort!, :sort_by!, :collect, :collect!, :map, :map!, :select, :select!, :keep_if, :values_at, :delete, :delete_at, :delete_if, :reject, :reject!, :zip, :transpose, :replace, :clear, :fill, :include?, :<=>, :slice, :slice!, :assoc, :rassoc, :+, :*, :-, :&, :|, :uniq, :uniq!, :compact, :compact!, :flatten, :flatten!, :count, :shuffle!, :shuffle, :sample, :cycle, :permutation, :combination, :repeated_permutation, :repeated_combination, :product, :take, :take_while, :drop, :drop_while, :bsearch, :pack]
{% endhighlight %}

もう一歩進んで、継承上位のクラスに特徴的なメソッド群がないか見たいこともあります。そんなときは、そのオブジェクトのクラス継承を調べて、メソッドを見つけることになります。

{% highlight ruby %}
irb> [1,2,3].class.ancestors
=> [Array, Enumerable, Object, Kernel, BasicObject]

irb> Enumerable.instance_methods(false)
=> [:to_a, :entries, :sort, :sort_by, :grep, :count, :find, :detect, :find_index, :find_all, :select, :reject, :collect, :map, :flat_map, :collect_concat, :inject, :reduce, :partition, :group_by, :first, :all?, :any?, :one?, :none?, :min, :max, :minmax, :min_by, :max_by, :minmax_by, :member?, :include?, :each_with_index, :reverse_each, :each_entry, :each_slice, :each_cons, :each_with_object, :zip, :take, :take_while, :drop, :drop_while, :cycle, :chunk, :slice_before, :lazy]
{% endhighlight %}

なんか...

面倒ですよね？

そんなわけで...Array#flattenに倣って...

<br />


`Object#methods`が数字を取れるようにしてみましたよ！

{% highlight ruby %}
module Skin
  def methods(lv=true)
    case lv
    when Fixnum
      return super(false) if lv<=0
      self.class.ancestors.take(lv)
          .map { |klass| klass.instance_methods(false) }
          .inject { |acc, klass| acc | klass }
    else
      super
    end
  end
end

Object.send(:include, Skin)
{% endhighlight %}


`0`はfalseと等価です。数字が増える度に継承上位のメソッド群が足されていきます。

{% highlight ruby %}
irb> [1,2,3].methods(0)
=> []

irb> [1,2,3].methods(1)
=> [:inspect, :to_s, :to_a, :to_ary, :frozen?, :==, :eql?, :hash, :[], :[]=, :at, :fetch, :first, :last, :concat, :<<, :push, :pop, :shift, :unshift, :insert, :each, :each_index, :reverse_each, :length, :size, :empty?, :find_index, :index, :rindex, :join, :reverse, :reverse!, :rotate, :rotate!, :sort, :sort!, :sort_by!, :collect, :collect!, :map, :map!, :select, :select!, :keep_if, :values_at, :delete, :delete_at, :delete_if, :reject, :reject!, :zip, :transpose, :replace, :clear, :fill, :include?, :<=>, :slice, :slice!, :assoc, :rassoc, :+, :*, :-, :&, :|, :uniq, :uniq!, :compact, :compact!, :flatten, :flatten!, :count, :shuffle!, :shuffle, :sample, :cycle, :permutation, :combination, :repeated_permutation, :repeated_combination, :product, :take, :take_while, :drop, :drop_while, :bsearch, :pack]

irb> [1,2,3].methods(2)
=> [:inspect, :to_s, :to_a, :to_ary, :frozen?, :==, :eql?, :hash, :[], :[]=, :at, :fetch, :first, :last, :concat, :<<, :push, :pop, :shift, :unshift, :insert, :each, :each_index, :reverse_each, :length, :size, :empty?, :find_index, :index, :rindex, :join, :reverse, :reverse!, :rotate, :rotate!, :sort, :sort!, :sort_by!, :collect, :collect!, :map, :map!, :select, :select!, :keep_if, :values_at, :delete, :delete_at, :delete_if, :reject, :reject!, :zip, :transpose, :replace, :clear, :fill, :include?, :<=>, :slice, :slice!, :assoc, :rassoc, :+, :*, :-, :&, :|, :uniq, :uniq!, :compact, :compact!, :flatten, :flatten!, :count, :shuffle!, :shuffle, :sample, :cycle, :permutation, :combination, :repeated_permutation, :repeated_combination, :product, :take, :take_while, :drop, :drop_while, :bsearch, :pack, :entries, :sort_by, :grep, :find, :detect, :find_all, :flat_map, :collect_concat, :inject, :reduce, :partition, :group_by, :all?, :any?, :one?, :none?, :min, :max, :minmax, :min_by, :max_by, :minmax_by, :member?, :each_with_index, :each_entry, :each_slice, :each_cons, :each_with_object, :chunk, :slice_before, :lazy]

irb> [1,2,3].class.ancestors
=> [Array, Enumerable, Object, Skin, Kernel, BasicObject]
irb> [1,2,3].methods(0).size
=> 0
irb> [1,2,3].methods(1).size
=> 89
irb> [1,2,3].methods(2).size
=> 120
irb> [1,2,3].methods(3).size
=> 120
irb> [1,2,3].methods(4).size
=> 121
irb> [1,2,3].methods(5).size
=> 160
irb> [1,2,3].methods(6).size
=> 167
irb> [1,2,3].methods.size
=> 167
{% endhighlight %}


Nokogiriでの結果はこんな感じです。

{% highlight ruby %}
irb> require 'nokogiri'
=> true
irb> html = Nokogiri::HTML('<html></html>')
=> #<Nokogiri::HTML::Document:0x3fd0ea1d71d0 name="document" children=[#<Nokogiri::XML::DTD:0x3fd0ea1d6e38 name="html">, #<Nokogiri::XML::Element:0x3fd0ea1d6ab4 name="html">]>

irb> html.class.ancestors
=> [Nokogiri::HTML::Document, Nokogiri::XML::Document, Nokogiri::XML::Node, Enumerable, Nokogiri::XML::PP::Node, Object, Skin, Kernel, BasicObject]

irb> html.methods(0)
=> []

irb> html.methods(1)
=> [:type, :meta_encoding, :meta_encoding=, :title, :title=, :serialize, :fragment]

irb> html.methods(2)
=> [:type, :meta_encoding, :meta_encoding=, :title, :title=, :serialize, :fragment, :root, :root=, :encoding, :encoding=, :version, :canonicalize, :dup, :url, :create_entity, :remove_namespaces!, :errors, :errors=, :create_element, :create_text_node, :create_cdata, :create_comment, :name, :document, :collect_namespaces, :decorators, :validate, :slop!, :decorate, :to_xml, :clone, :namespaces, :add_child, :<<, :to_java]

irb> html.methods(3)
=> [:type, :meta_encoding, :meta_encoding=, :title, :title=, :serialize, :fragment, :root, :root=, :encoding, :encoding=, :version, :canonicalize, :dup, :url, :create_entity, :remove_namespaces!, :errors, :errors=, :create_element, :create_text_node, :create_cdata, :create_comment, :name, :document, :collect_namespaces, :decorators, :validate, :slop!, :decorate, :to_xml, :clone, :namespaces, :add_child, :<<, :to_java, :add_namespace_definition, :node_name, :node_name=, :parent, :child, :first_element_child, :last_element_child,
:children, :element_children, :next_sibling, :previous_sibling, :next_element, :previous_element, :node_type, :content, :path, :key?, :namespaced_key?, :blank?, :attribute_nodes,
 :attribute, :attribute_with_ns, :namespace, :namespace_definitions, :namespace_scopes, :encode_special_chars, :unlink, :internal_subset, :external_subset, :create_internal_subse
t, :create_external_subset, :pointer_id, :line, :native_content=, :decorate!, :search, :/, :xpath, :css, :>, :at, :%, :at_xpath, :at_css, :[], :[]=, :add_previous_sibling, :add_n
ext_sibling, :before, :after, :inner_html=, :children=, :replace, :swap, :next, :previous, :next=, :previous=, :remove, :get_attribute, :attr, :set_attribute, :text, :inner_text,
 :has_attribute?, :name=, :to_str, :elements, :attributes, :values, :keys, :each, :remove_attribute, :delete, :matches?, :parse, :content=, :parent=, :comment?, :cdata?, :xml?, :
html?, :text?, :fragment?, :description, :read_only?, :element?, :elem?, :to_s, :inner_html, :css_path, :ancestors, :default_namespace=, :add_namespace, :namespace=, :traverse, :
accept, :==, :to_html, :to_xhtml, :write_to, :write_html_to, :write_xhtml_to, :write_xml_to, :<=>, :do_xinclude]

irb> html.methods(3).size
=> 142
irb> html.methods(4).size
=> 190
irb> html.methods(5).size
=> 192
irb> html.methods.size
=> 231
{% endhighlight %}


どうでしょうか。って、お前の分別はどうしたって話ですけど。

そういえば、`Object.methods`の引数ってどんなオブジェクトも受けるんですねー。`false`, `nil`以外は全部`true`と解釈されますね。

---

(追記：2013-09-26)

Twitterで@niku_nameさんに、`obj.public_methods(false)`を教えて頂きました。これはobjにシングルトンメソッドが定義されていなければ、obj.class.instance_methods(false)と同じになります。

> [Twitter / niku_name: 僕は obj.public_methods(false) ...](https://twitter.com/niku_name/status/383074490726363136 "Twitter / niku_name: 僕は obj.public_methods(false) ...")

(追記：2013-09-26)

Twitterで@mad_pさんに、pryでcdしてlsする方法を教えて頂きました。

> [Twitter / mad_p: 調べるだけなら、 pry ...](https://twitter.com/mad_p/status/383067544292032512 "Twitter / mad_p: 調べるだけなら、 pry ...")

nokogiriの例で見てみます。`cd [obj] => ls`とせずに、`ls [obj]`としてもいいのですね。モジュール別にメソッドがリスティングされるので、これは見やすくて便利！

{% highlight bash %}
[1] pry(main)> html = Nokogiri::HTML('<html></html>')
=> #(Document:0x3fe855ebaa3c {
  name = "document",
  children = [
    #(DTD:0x3fe855eba0c8 { name = "html" }),
    #(Element:0x3fe855ebfb18 { name = "html" })]
  })

[2] pry(main)> html.class.ancestors
=> [Nokogiri::HTML::Document,
 Nokogiri::XML::Document,
 Nokogiri::XML::Node,
 Enumerable,
 Nokogiri::XML::PP::Node,
 Object,
 PP::ObjectMixin,
 Kernel,
 BasicObject]

[3] pry(#<Nokogiri::HTML::Document>):1> ls html
Nokogiri::XML::PP::Node#methods: inspect  pretty_print
Enumerable#methods:
  all?            each_cons         flat_map  min           select
  any?            each_entry        grep      min_by        slice_before
  chunk           each_slice        group_by  minmax        sort
  collect         each_with_index   include?  minmax_by     sort_by
  collect_concat  each_with_object  inject    none?         take
  count           entries           lazy      one?          take_while
  cycle           find              map       partition     to_a
  detect          find_all          max       reduce        zip
  drop            find_index        max_by    reject
  drop_while      first             member?   reverse_each
Nokogiri::XML::Node#methods:
  %                       delete                node_name=
  /                       description           node_type
  <=>                     do_xinclude           parent=
  ==                      each                  parse
  >                       elem?                 path
  []                      element?              pointer_id
  []=                     element_children      previous
  accept                  elements              previous=
  add_next_sibling        encode_special_chars  previous_element
  add_previous_sibling    external_subset       previous_sibling
  after                   first_element_child   read_only?
  ancestors               fragment?             remove
  at                      get_attribute         remove_attribute
  at_css                  has_attribute?        replace
  at_xpath                html?                 search
  attr                    inner_html            set_attribute
  attribute               inner_html=           text
  attribute_nodes         inner_text            text?
  attribute_with_ns       internal_subset       to_html
  before                  key?                  to_s
  blank?                  keys                  to_str
  cdata?                  last_element_child    to_xhtml
  child                   matches?              traverse
  children                name=                 unlink
  children=               namespace=            values
  comment?                namespace_scopes      write_html_to
  content                 namespaced_key?       write_to
  content=                native_content=       write_xhtml_to
  create_external_subset  next                  write_xml_to
  create_internal_subset  next=                 xml?
  css                     next_element          xpath
  css_path                next_sibling
  decorate!               node_name
Nokogiri::XML::Document#methods:
  <<                  create_comment    document   name                to_java
  add_child           create_element    dup        namespaces          to_xml
  canonicalize        create_entity     encoding   remove_namespaces!  url
  clone               create_text_node  encoding=  root                validate
  collect_namespaces  decorate          errors     root=               version
  create_cdata        decorators        errors=    slop!
Nokogiri::HTML::Document#methods:
  fragment  meta_encoding  meta_encoding=  serialize  title  title=  type
self.methods: __pry__
instance variables: @decorators  @errors  @node_cache
{% endhighlight %}
