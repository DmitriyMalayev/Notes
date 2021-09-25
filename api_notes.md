`--api flag`
To create an API-only Rails build, include the --api after the name of the Rails app name upon creation 
rails new bird-watcher-api --api

Controllers inherit from ActionController::API 

`Create 3 Migrations, Models, and Containers`
rails g resource bird name species
rails g resource location latitude longitude 
rails g resource sighting bird:references location:references 

```rb           
#Seed File
bird_a = Bird.create(name: "Black-Capped Chickadee", species: "Poecile Atricapillus")
bird_b = Bird.create(name: "Grackle", species: "Quiscalus Quiscula")
bird_c = Bird.create(name: "Common Starling", species: "Sturnus Vulgaris")
bird_d = Bird.create(name: "Mourning Dove", species: "Zenaida Macroura")
 
location_a = Location.create(latitude: "40.730610", longitude: "-73.935242")
location_b = Location.create(latitude: "30.26715", longitude: "-97.74306")
location_c = Location.create(latitude: "45.512794", longitude: "-122.679565")
 
sighting_a = Sighting.create(bird: bird_a, location: location_a)
sighting_b = Sighting.create(bird: bird_b, location: location_b)
sighting_c = Sighting.create(bird: bird_c, location: location_c)

# Controller Actions 
def index
    sightings = Sighting.all
    render json: sightings, include: [:bird, :location]
end 
```
# Cross-Origin Resource Sharing (CORS)
While working on your APIs you'll want to have your Rails server running while also trying our various endpoints using fetch()
CORS is designed to prevent scripts like fetch() from one origin accessing a resource from a different origin unless that resource specifically states that it expects to share. 