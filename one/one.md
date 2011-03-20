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

