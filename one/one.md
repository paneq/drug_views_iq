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
        <th><%= Post.human_attribute_name(:title) %></th>
        <th><%= Post.human_attribute_name(:body) %> </th>
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


!SLIDE code smaller
# posts_controller.rb
    @@@ ruby
    class PostsController < ApplicationController

      def index
        @posts = Post.all
      end
      def show
        @post = Post.find(params[:id])
      end
      def new
        @post = Post.new
      end
      def edit
        @post = Post.find(params[:id])
      end
      def create
        @post = Post.new(params[:post])

        if @post.save
          redirect_to(@post, :notice => 'Post was successfully created.')
        else
          render :action => :new
        end
      end
      def update
        @post = Post.find(params[:id])
        if @post.update_attributes(params[:post])
          redirect_to(@post, :notice => 'Post was successfully updated.')
        else
          render :action => :edit
        end
      end
      def destroy
        @post = Post.find(params[:id])
        @post.destroy
        redirect_to(posts_url)
      end
    end

!SLIDE

* Pierwsza zmiana
* Niech nasz model będzie w namespace "Cms"
* Ile trzeba będzie zmienić w widokach i kontrolerze ?

!SLIDE

* Kontroler - 7 linii
* Widok - 2 linie (tyle ile razy human_attribute_name by znać tytuł nagłówka dla tabelki)
* Widok & Kontroller - 9 linii - przestał działać routing... (undefined method `cms_post_path')

!SLIDE code smaller
# posts_controller.rb
    @@@
    -    @post = Cms::Post.new(params[:post])
    +    @post = Cms::Post.new(params[:cms_post])


    -      redirect_to(@post)
    +      redirect_to( post_url(@post) )

    -<%= render 'form' %>
    +<%= form_for(@post, :url => post_path(@post) ) do |f| %>
    +  <%= render :partial => 'form', :locals => {:f => f} %>
    +<% end %>

    -<%= link_to 'Show', @post %> |
    +<%= link_to 'Show', post_path(@post) %> |

