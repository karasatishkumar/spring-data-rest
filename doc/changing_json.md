# Customizing the JSON output

Sometimes in your application you need to provide links to other resources from a particular entity. For example, a
`Customer` response might be enriched with links to a current shopping cart, or links to manage resources related to
that entity. Spring Data REST provides integration with [Spring HATEOAS](https://github.com/SpringSource/spring-hateoas)
and provides an extension hook for users to alter the representation of resources going out to the client.

### The ResourceProcessor interface

Spring HATEOAS defines a `ResourceProcessor<?>` interface for processing entities. All beans of type
`ResourceProcessor<Resource<T>>` will be automatically picked up by the Spring Data REST exporter and triggered when
serializing an entity of type `T`. For example, to define a processor for a `Person` entity, add a `@Bean` to your
ApplicationContext like the following (which is taken from the Spring Data REST tests):

		@Bean public ResourceProcessor<Resource<Person>> personProcessor() {
			return new ResourceProcessor<Resource<Person>>() {
				@Override public Resource<Person> process(Resource<Person> resource) {
					resource.add(new Link("http://localhost:8080/people", "added-link"));
					return resource;
				}
			};
		}


### Adding Links

It's possible to add links to the default representation of an entity by simply calling `resource.add(Link)` like the
example above. Any links you add to the `Resource` will be added to the final output.

### Customizing the representation

The Spring Data REST exporter executes any discovered `ResourceProcessor`s before it creates the output representation.
It does this by registering a `Converter<Entity, Resource>` instance with an internal `ConversionService`. This is the
component responsible for creating the links to referenced entities (e.g. those objects under the "links" property in
the object's JSON representation). It takes an `@Entity` and iterates over its properties, creating links for those
properties that are managed by a `Repository` and copying across any embedded or simple properties.

If your project needs to have output in a different format, however, it's possible to completely replace the default
outgoing JSON representation with your own. If you register your own `ConversionService` in the ApplicationContext and
register your own `Converter<Person, Resource>`, then you can return a `Resource` implementation of your choosing.
