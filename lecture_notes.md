Nested Resources

A concept in RAILS called a NESTED RESOURCE
Common pattern in an application.
After this lecture you should be able to
   create a nested resource
   create and edit the resources
   
   
So typically for a basic Restful site you would have routes like:
/users
/users/new
/profile
/profile/new

/users/1
/users/1/edit
/profile/1
/profile/1/edit

But what if we want the profile to belong to a user?
We want a route like
/users/1/profile

This makes sense Because a profile will never exist without a user. 



----------------------------------------------------------------------------------------


Here is our site. a portfolio site.
You are a designer or someone creative and you want to create a portfolio. The portfolio will have your name and about you. But you also want to add projects to your portfolio.

One way to add related data is through a nested form.
(go to edit portfolio)
So here we would edit a portfolio 
you can imagine on this form to have add a project
  maybe something like Project 1 project 2, 3, etc.
but then the only way to add a project to a portfolio is by editing the portfolio itself.

Conceptually we have these 2 resources, we have Portfolios and Projects.
  we don;t always want to edit a portfolio when we are only adding a project to it.
  
So we want to create a form that relates to a portfolio but is only about adding a project.



URL:
So lets look about the url
  GO TO PROJECTS/NEW
  
  if we went to projects/new why does this now work for mapping to a specific portfolio?
  
  The idea of adding a project to a portfolio website outside of the context of a specific portfolio doesn't make sense. You would have this random project in your database the has no portfolio.
  
  So using NESTED RESOURCES we can have a URL that looks like:
  portfolios/1/PROJECTS/NEW
  
  SO in this ONE url we have all the data we need to relate the project that we are adding to the system to a particular portfolio.
  
  This projects new form becomes super simple. It is just a projects form, it doesnt even mention the portfolios fields.
  
  
GO TO PORTFOLIO/1: 
  So we want to put a link here to add new projects.
  
  and on the portfolio show page we will list out all your projects with a link to edit them. We are going to build these CRUD features for a project in a portfolio. and the interface is going to come from the portfolio show page.
  
  It is very realistic. any site that has multiple images like Air BNB on your apt listing, you have a button to add another image, this is where nested resources comes in handy.
  
  Use it alot in wizards or steps. fill out your address then credit card or fill out section 1, then section 2. This is where you would want to express nested resources. So it can be like 'accounts/1/addresses/new'
  
  
  
  The only thing I have in this APPLICATION IS a portfolio model. It is all scaffolded.


### add a link to the portfolios show page. stub out
link_to "add Project" 'portfolio/1/projects/new'

rake routes

SO we want this path but our routes do not have that
  How would we do this? Where do we reference our routes?
  
config/routes.rb

we currently have 
  resources :portfolios
  
  resources :projects
  
  So we want to nest them. To do so we just nest them :)

  resources :portfolios do 
    resources :projects
  end

  And you can nest them as many times as you want.
  resources :portfolios do 
    resources :projects do
      resources :images
    end
  end
  now you have images related to a project related to a portfolio.
  
  
  So now lets look at what the routes look like now that we generated this.
  
 rake routes

look at the routes.
The parent element gets prefixed automatically and the id route variable is reserved for projects.

now in params you would access the portfolio's id from portfolio_id and projects id by id.


So lets use this new_portfolio_project_path and this route has how many variables? What would we pass?
the portfolio that we want to add this project to.

add the path to the portfolio show page.

Then click the button.

show error.
Mention that it wants ProjectsController. Since the routes are nested and the resources are nested, the controller does not need to be nested.

So we are going to build a new projects controller.

rails g resource project

So first it added the top level route. Lets remove this. We will never have a project out of context of a portfolio.

MIGRATION
so give me fields of what a project may have?
t.string :title
t.text :description
t.integer :portfolio_id


Now relations
portfolio has_many :projects
projects belongs_to :portfolio




So now that we have our schema set up, lets build our action.

Projects Controller

def new

end

So what is the requirement for this new action. We are trying to create a new project in the context of a portfolio. 
We need a portfolio id
  @portfolio = Portfolio.find(params[:portfolio_id])
  @project = @portfolio.projects.build

So now lets build out our NEW view.

new.html.erb


<h1> Add a Project to <%= @portfolio.name.capitalize %> <h1>

<%= form_for(@project) do |f| %>


<% end %>


So now go to the form on browser.

IT IS BREAKING! Why is it breaking?

projects_path is breaking, why is this happening? I dont even see where I reference the path!?

we need to give the form an action. Since rails is magical, it tries to autoguess the path and by default since the form is for a project it defaulted to the conventional projects_path,
so how do I tell this form where to submit to?!?!?!

<%= form_for(@project, :url => '/') do |f| %>
what route do we want this new project to submit to?


<%= form_for(@project, :url => portfolio_projects_path(@portfolio) do |f| %>


inspect the element and show the action.
This is restful. 


GOALS
Now we would just build this form
the create action
show action
edit action
update action

<p>
<%= f.label :title %>
<%= f.text_field :title %>
</p><p>
<%= f.label :description %>
<%= text_field :description %>
</p>



So what do we think the params name attribute will be for this text_field?
What options do we think there are? will it be nested?
portfolios[projects][title]
or
projects[title]


it is projects[title] INSPECT ELEMENT AND SHOW
Singular form object. There is no nesting in the form.
Only the resource is nested.

submit the form . let error happen

show params

now 
def create

end


So given params, what do we want to do here?
@portfolio = Portfolio.find(params[:portfolio_id])
@project = @portfolio.projects.build(project_params)
if @project.save
  redirect_to @portfolio
  else
  render 'new'
  end

add
private
def project_params
  params.require(:project).permit(:title, :description)
end


PORTFOLIO SHOW PAGE.
@portfolio.projects.each do |project| 
link_to title, portfolio_project_path(@portfolio, project)
end





this way of doing the forms is way easier than doing a nested fields_for



Now show action we will build out

def show

end
What data do I need for show?
@portfolio = Portfolio.find(params[:portfolio_id])

so we did this line of code a few times already, is there anyway we can make this a method?

before_action :set_portfiolio
def set_portfilio
  @portfolio = Portfolio.find(params[:portfolio_id])
end

DELETE THOSE LINES FOR EVERYTHING ELSE.

now I can just do

@project = @portfolio.projects.find(params[:id])

So WHY do I find the project throught the Portfolio rather than just through Projects.find?

We want to scope our search query down. Only projects that belong to this portfolio.


<h1><%= @project.title %>, in <%= @project.portfolio.name %> /h1>

<p><%= @project.description %></p>

<p>
HOW DO WE WANT TO LINK TO THIS PROJECT? how many variables do we need?
  <%= link_to "Edit #{@project.title}", edit_portfolio_project_path(@portfolio, @project) %>
</p>



Now we are going to do our edit form.
Edit form
I want to build this and reuse my new form

create a form partial

move the form over.

<%= render 'form' %>

create an edit view. copy entire new page and paste there.

change to Edit. title
Edit a <%= @project.title.capitalize %>

<%= render 'form' %>


So now lets build the Edit controller.
So what Do I need to edit a project in a portfolio?

get the project.

def edit
  @project = @portfolio.projects.find(params[:id])
end


So it should be working......
So if I submit it, it gives me a route error. But we used the correct routing. So I have this route, so why am I getting this error? The route exists.
rake routes.
What is this route missing?
missing the project id

The form says to use portfolio_projects_path(@portfolio) 
THIS is tough.... if we are coding the form for like this we cannot share the form between new and edit since they both go to different places.

So I like to seperate our form envelope from the fields. So lets rename the partial fields. and leave it to the view to create the form wrapper.

So in edit you have:
<%= form_for(@project, :url => portfolio_project_path(@portfolio, @project)) do |f| %>
<%= render 'fields', :f => f %>
<% end %>


can take this in new also.
<%= form_for(@project, :url => portfolio_projects_path(@portfolio, @project)) do |f| %>
<%= render 'fields', :f => f %>
<% end %>




---- def update
raise params
end

show params that we now have the id and portfolio_id


So lets build out the update
  def update
    @project = @portfolio.projects.find(params[:id])
    @project.update(project_params)
    redirect_to @portfolio
  end























