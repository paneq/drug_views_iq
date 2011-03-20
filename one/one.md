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

!SLIDE bullets

* Druga zmiana
* Niech nasz kontroler będzie w namespace "Admin"
* Ile trzeba będzie zmienić w widokach i kontrolerze ?

!SLIDE bullets

* 14 linii pasujących do wyrażenia /.*post.*(_url|_path)/
* Gdybybm chciał zrobić by całość działała jako podresource wewnątrz :users i tylko widzieć
wiadomości danego autora też musiałbym zmienić mniej więcej tyle linii kodu i wszędzie do generatorów
urli podstawić dodatkowy parametr @user. Niefajnie...


!SLIDE bullets small
# Co jest zatem problemem w naszych widokach zwykle ?
* duża wrażliwość kodu na zmiany namespace. Zarówno modelu, kontrolera jak i routingu.
* w domyślnym scaffoldzie jest bardzo dużo powtórzeń
* w domyślnym scaffoldzie po niektórych zmianach część rzeczy kodu je odwzoruje np formularze będą generować :cms_post
a część kodu "nie nadąży za tą zmianą".
*

!SLIDE bullets
# Co jest największym problem ?
* Routing
* bardzo fajny
* bardzo konfigurowalny, elastyczny
* mogę mieć 2 ścieżki pod jeden kontroler
* mogę mieć 2 kontrolery pod jedną ścieżką (w zależności od constraint'ów)
* ale w 95% przypadków jest to proste mapowanie 1-1 bez zbędnych gadżetów...

!SLIDE small center
# Jak to dawniej bywało ?
# Rails 1 anybody ?
    @@@ Ruby
    link_to 'Show',
    :action => :show,
    :id => post.id

!SLIDE small center
# Rails 1
# zagnieżdżone resource'y
    @@@ Ruby
    link_to 'Show',
    :action => :show,
    :user_id => post.author_id,
    :id => post.id

!SLIDE bullets
# Problemy ?
* Komu by się chciało to pisać ?
* Wciąż mnóstwo powtórzeń, każdy link scope'owany względem nadrzędnego resource'u
* Czemu mój widok miałoby to w ogóle obchodzić ?

!SLIDE bullets
# Co jest największym marzeniem ?
* Gdybym mógł jak w Rails 1 powiedzieć, że chcę dostać się do tej akcji (bo tak o tym myślę i nie obchodzi mnie cały ten routingowy narzut)
* Gdybym nie musiał się powtarzać

!SLIDE bullets
# Open source pozwala spełniać marzenia :-)
* Ale to co pokaże później jeszcze nie jest open :-(

!SLIDE bullets

* Załóżmy, że mamy jakiś bardziej złożony kontroler, w którym mogę sterować czy na layoucie
mam coś wyświetlać czy nie. Niech w naszym przykładzie będzie to kalendarz który pozwala mi
zobaczyć w jakie dni coś napisałem na blogasku i ewentualnie przejść w to miejsce.
Nie będziemy tego szczegółowo implementować. Jedyne co mnie interesuje to ustawianie
poszczególnych zmiennych potrzebnych kalendarzowi do odrysowania się.
Powiedzmy, że potrzebuje on miesiąc i rok znać by zadziałać.

!SLIDE small
# Kontroler
    @@@ Ruby
    module Admin
      class PostsController < ApplicationController

        before_filter :setup_calendar

        ...

        private

        def setup_calendar
          @calendar = rand > 0.5 # Czytanie z params albo session albo z ustawień ...
          if @calendar
            @year = (params[:year] || Date.today.year).to_i
            @month = (params[:month] || Date.today.month).to_i
            @count = @year / @month
          end
        end

      end
    end

!SLIDE smaller code
# Layout
    @@@ Ruby
    <% if @calendar %>
      <div class="calendar">
        W miesiącu <%= @year %> - <%= @month %> napisano
        <%= @count %> wiadomości

        <%= link_to "Poprzedni miesiąc",
        admin_posts_path(:year => @year, :month => @month - 1)  %>

        <%= link_to "Następny miesiąc",
        admin_posts_path(:year => @year, :month => @month + 1)  %>

      </div>
    <% end %>

!SLIDE bullets
# Co tutaj jest problematyczne ?
* Powtórzenie logiki ustawiania i korzystania ze zmiennych
* Musze w kontrolerze naprzód wiedzieć co zostanie użyte i to ustawić
* kombinacji takich możliwości może być dużo (w przykładzie były tylko dwie)


!SLIDE
# Must refactor!

!SLIDE bullets
# Co tutaj jest problematyczne ?
* https://github.com/voxdolo/decent_exposure is your friend