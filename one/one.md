!SLIDE title-slide
# Jakie jest IQ twoich widoków ? #

!SLIDE title-slide
# W tym odcinku będziemy stawać się madrzejsi chyba.

!SLIDE bullets
* Robert Pankowecki
* http://robert.pankowecki.pl/

!SLIDE bullets incremental
# Co zrobimy ?
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

!SLIDE bullets incremental
# Pierwsza zmiana
* Niech nasz model będzie w namespace "Cms"
* Ile trzeba będzie zmienić w widokach i kontrolerze ?

!SLIDE bullets incremental
# Zmienione linie
* Kontroler - 7 linii
* Widok - 2 linie (tyle ile razy human attribute name wywołany by znać tytuł nagłówka dla tabelki)
* Widok & Kontroller - 9 linii - przestał działać routing...

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

!SLIDE bullets incremental
# Druga zmiana
* Niech nasz kontroler będzie w namespace "Admin"
* Ile trzeba będzie zmienić w widokach i kontrolerze ?

!SLIDE bullets

* 14 linii pasujących do wyrażenia /.*post.*(url|path)/
* Gdybybm chciał zrobić by całość działała jako podresource wewnątrz :users i tylko widzieć
wiadomości danego autora też musiałbym zmienić mniej więcej tyle linii kodu i wszędzie do generatorów
urli podstawić dodatkowy parametr @user. Niefajnie...


!SLIDE bullets small incremental
# Co jest zatem problemem w naszych widokach zwykle ?
* duża wrażliwość kodu na zmiany namespace. Zarówno modelu, kontrolera jak i routingu.
* w domyślnym scaffoldzie jest bardzo dużo powtórzeń
* po zmianach część kodu je odwzoruje np. formularze będą generować :cms_post
a część kodu "nie nadąży za tą zmianą" bo były wygenerowane a nie są dynamicznie obliczane

!SLIDE bullets small incremental
# Co jest największym problem ?
* Routing
* bardzo fajny, konfigurowalny, elastyczny
* mogę mieć 2 ścieżki pod jeden kontroler
* mogę mieć 2 kontrolery pod jedną ścieżką (w zależności od constraint'ów)
* ale w 95.95 % przypadków jest to proste mapowanie 1-1 bez zbędnych gadżetów...

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

!SLIDE bullets incremental
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
* Ale to co pokaże później, jeszcze nie jest open :-(

!SLIDE bullets
# Zanim fajerwerki
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

!SLIDE bullets incremental
# Co tutaj jest problematyczne ?
* Powtórzenie logiki ustawiania i korzystania ze zmiennych
* Musze w kontrolerze naprzód wiedzieć co zostanie użyte i to ustawić
* kombinacji takich możliwości może być dużo (w przykładzie były tylko dwie)


!SLIDE
# Must refactor!

!SLIDE bullets
# Love!
* https://github.com/voxdolo/decent_exposure is your friend

!SLIDE smaller code
# Kontroler
    @@@ Ruby
      class PostsController < ApplicationController

        expose(:model) { Cms::Post }
        expose(:posts) { model.scoped }
        expose(:post)
        expose(:as) { :post }

        def create
          if post.save
            redirect_to(admin_post_url(post), :notice => 'Post was successfully created.')
          else
            render :action => :new
          end
        end
        def update
          if post.save
            redirect_to(admin_post_url(post), :notice => 'Post was successfully updated.')
          else
            render :action => :edit
          end
        end
        def destroy
          post.destroy
          redirect_to(admin_posts_url)
        end
      end

!SLIDE smaller code
# Domyślne zachowanie decent:
    @@@ Ruby
      klass.default_exposure do |name|
        collection = name.to_s.pluralize
        if respond_to?(collection) && collection != name.to_s && send(collection).respond_to?(:scoped)
          proxy = send(collection)
        else
          proxy = name.to_s.classify.constantize
        end

        if id = params["#{name}_id"] || params[:id]
          proxy.find(id).tap do |r|
            r.attributes = params[name] unless request.get?
          end
        else
          proxy.new(params[name])
        end
      end

!SLIDE bullets
# Co zyskaliśmy ?
* Odporność na zmiany namespace modelu

!SLIDE smaller code
# Kontroler
    @@@ Ruby
      class PostsController < ApplicationController

        expose(:calendar?) { rand > 0.5 } # Czytanie z params albo session albo z ustawień ...
        expose(:year)  { (params[:year] || Date.today.year).to_i }
        expose(:month) { (params[:month] || Date.today.month).to_i }
        expose(:count) { year / month }

      end

!SLIDE bullets incremental
# Co zyskaliśmy ?
* Kontroler mówi co może dać a co z tego weźmiesz na widoku to twoja sprawa
* Jeśli akurat jakiś danych nie potrzebujesz z powodu ustawień/decyzji użytkownika to żaden problem i po prostu nie wywoła się dostępna w kontrolerze metoda
* Jeśli jedne dane wymagają pobrania innych to dalej żaden problem, po prostu znów zostanie wywołana odpowiednia metoda. (przykład :count)

!SLIDE bullets
# Czego nam brakuje ?
* Ochrony przed zmianamy w routingu/kontrolerze

!SLIDE smaller code
# ApplicationController
    @@@ Ruby
      def self.inherited(subclass)
        super
        subclass.send(:reveal_short_routes)
      end

      def self.reveal(name, &block) # Nie cachuje wyniku w przeciwieństwie do expose oraz może przyjmować parametry
        define_method(name, &block)
        helper_method(name)
        protected(name)
        hide_action(name)
      end

      def self.reveal_short_routes
        p = controller_path
        _routes.routes.
        select{|r| r.defaults[:controller] == p}.
        each do |r|
          next unless r.name
          reveal(:"#{r.defaults[:action]}_route") do |object = nil|
            send(:"#{r.name}_path", routing_mapper(r, object) )
          end
        end
      end
      def self.route_action?(route, action)
        route.defaults[:action].to_sym == action.to_sym
      end

!SLIDE smaller code
# ApplicationController
    @@@ Ruby
      # Dodatkowa konwencja
      reveal(:entry_route) do |obj = nil|
        show_route(obj)
      end if route_action?(r, :show)

      reveal(:collection_route) do
        index_route
      end if route_action?(r, :index)

      reveal(:collection_route) do
        create_route
      end if route_action?(r, :create)


!SLIDE smaller code
# Przykład : AddressController
# jako zagnieżdżony resource usera
    @@@ Ruby
      def routing_mapper(route, object = nil) # Objec używany na widokach np index.
        h = Hash.new
        if route.segment_keys.include?(:user_id)
          h[:user_id] = user
        end
        if route.segment_keys.include?(:id)
          h[:id] = (object || address)
        end
        return h
      end


!SLIDE smaller code
# Kontroler
    @@@ Ruby
    def create
      if post.save
        redirect_to(entry_route, :notice => 'Post was successfully created.')
      else
        render :action => :new
      end
    end

    def update
      if post.save
        redirect_to(entry_route, :notice => 'Post was successfully updated.')
      else
        render :action => :edit
      end
    end

    protected

    def routing_mapper(route, object = nil) # Objec używany na widokach np index.
      h = Hash.new
      if route.segment_keys.include?(:id)
        h[:id] = (object || post)
      end
      return h
    end

!SLIDE smaller code
# Widoki
    @@@
    # Index
    <% posts.each do |post| %>
      <tr>
        <td><%= post.title %></td>
        <td><%= link_to 'Show', entry_route(post) %></td>
        <td><%= link_to 'Edit', edit_route(post) %></td>
        <td><%= link_to 'Destroy', entry_route(post), :method => :delete %></td>
      </tr>
    <% end %>

    # Edit
    <%= form_for(post, :as => as, :url => entry_route ) do |f| %>
      <%= render :partial => 'form', :locals => {:f => f} %>
    <% end %>
    <%= link_to 'Show', entry_route %> |
    <%= link_to 'Back', collection_route %>

    # New
    <%= form_for(post, :as => as, :url => collection_route) do |f| %>
      <%= render :partial => 'form', :locals => {:f => f} %>
    <% end %>

!SLIDE bullets incremental
# Czy to wszystko miało sens ?
* Usuńmy namespace modelu, kontrolera i routingu!
* 3 linijki trzeba było zmienić :-)
* a przy drobnych poprawkach wystarczyłaby jedna! Zmiana nazwy modelu w kontrolerze!
