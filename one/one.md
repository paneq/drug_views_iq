!SLIDE title-slide
# Jakie jest IQ twoich widoków ? #

!SLIDE bullets
* Robert Pankowecki
* http://robert.pankowecki.pl/

!SLIDE

* Zaczniemy od dobrze znanego nam kodu
* I spróbujemy go poruszyć w różne strony by zobaczyć co się z nim stanie i jak będzie reagował na zmiany


!SLIDE command
# rails g
# scaffold 
# Post
# title:string
# body:text
# author_id:integer


!SLIDE code smaller
# index.html.erb
    @@@ ruby

    <table>
      <tr>
        <th>Post.human_attribute_name(:title) </th>
        <th>Post.human_attribute_name(:body)  </th>
        <th>Post.human_attribute_name(:author)</th>
      </tr>

    <% @posts.each do |post| %>
      <tr>
        <td><%= post.title %> </td>
        <td><%= post.body %>  </td>
        <td><%= post.author %></td>
        <td>
            <%= link_to 'Show', post %>
            <%= link_to 'Edit', edit_post_path(post) %>
            <%= link_to 'Destroy', post, :confirm => 'Are you sure?', :method => :delete %>
        </td>
      </tr>
    <% end %>
    </table>
    <%= link_to 'New Post', new_post_path %>
