# Zitec FormAutocompleteBundle
This bundle creates autocomplete form fields for you entities.

## Instalation
  * composer require zitec/form-autocomplete-bundle
  * Add routing for autocmplete in config/routes/zitec_autocomplete.yaml:
    ```yaml
    zitec_form_autocomplete:
        resource: "@FormAutocompleteBundle/Resources/config/routing.yml"
            prefix:   /
    ```
  * Add template fields in twig.yml:
    ```yaml
    twig:
        form:
            resources:
                - 'FormAutocompleteBundle:Form:fields.html.twig'
    ```
  * add js and css in template
    *  select2 library
    *  bundles/zitecformautocomplete/css/autocomplete.css
    * @FormAutocompleteBundle/Resources/public/js/autocomplete.js
    * @FormAutocompleteBundle/Resources/public/js/autocomplete_init.js

## How to use
  * declare service used for handling the data of city autocomplete fields
    ```yaml
    campaigns.form.autocomplete.data_resolver_cities_with_campaigns:
        class: Zitec\FormAutocompleteBundle\DataResolver\EntitySingleDataResolver
        arguments:
            - @doctrine
            - GeolocationsBundle\Entity\City
            - id
            - name
            - getCityWithNameLike
        tags:
            - { name: zitec_autocomplete_data_resolver, key: cities_with_campaigns_single }
    ```
  * in city repository create function getCityWithNameLike
    ```php
        public function getCityWithNameLike($cityName)
            {
                $queryBuilder = $this->createQueryBuilder('c')
                        ->where('c.name like :name or c.internationalName like :name')
                        ->orderBy('c.name', 'ASC')
                        ->setParameter('name', '%'.$cityName.'%');
                //fetch matching cities
                $cities = $queryBuilder->getQuery()->getResult();
                return $cities;
            }
    ```
  * in form create autocomplete field
    ```php
        ->add('city', 'zitec_autocomplete', array(
            'data_resolver' => 'cities_with_campaigns_single',
            'placeholder' => 'placeholder_campaign_list_city',
            'required' => false,
            'delay' => 250,
            'allow_clear' => true,
        ))
    ```
    
## Components

### AutocompleteController
Controller which handles autocomplete specific actions.

#### Methods
  * indexAction(Request $request, $dataResolverId) : internal action which provides autocomplete suggestions specific to the given data resolver;
  * parametersIsValid($parameter): validate that the parameter we are receiving have the proper data types.The parameter must be a scalar value or empty.

### DataResolverInterface
A data resolver is an object which manages the data of an autocomplete field. When a programmer attaches an autocomplete field to a form, it must
also specify a data resolver for it. It should be able to:
  * suggest items matching the user's search criteria;
  * transform the user input into application data;
  * transform the application data into view data (the reverse of the preceding operation);
#### Methods
  * getSuggestions($term, $context = null): Given the user search term, returns a list of matching suggestions.
    * @param: string $term
    * @param: mixed $context: when demanding autocomplete suggestions, the client may also specify a context which can influence the result generation;
    * @return: array - a set of arrays or objects (which can be JSON-serialized) with the following keys:
      * id: the identifier of the suggested item;
      * text: the label of the suggested item;
  * getModelData($viewData, $viewDataAlwaysString = false): Extracts the model data (the data used in the application) from the view data.
    * @param mixed $viewData
    * @param bool $viewDataAlwaysString: flag which specifies that the data received from the client will always be represented as a string,
      event if the field carries multiple values.
    * @return mixed
  * getViewData($modelData): Extracts the view data (that will be used in the views) from the model data.
    * @param: mixed $modelData
    * @return: mixed – the data in the view should be represented as an array or a collection of arrays with the following keys:
      * Value: the actual data;
      * Label: a description of data;

### DataResolverManager
Manages the autocomplete data resolvers declared throughout the application.
#### Fields
  * $dataResolver: the collection of managed data resolvers keyed by their identifiers.
#### Methods
  * Get($key): Fetches the data resolver with the given key.

### EntityBaseDataResolver

#### Fields
  * $doctrine: doctrine service;
  * $entityClass: the associated entity’s class;
  * $idPath: the path to the id property of the entity;
  * $labelPath: the path to the entity property which represents its label;
  * $suggestionsFetcher: the consumer may provide a custom function for fetching the suggestions data. The function will receive the term
  and should return an array of matching entities of the specified type. It will be represented in one of the forms:
    * a simple string: denotes the name of a method from the entity repository;
    * a callable: denotes the complete path to a function;
  * $propertyAccesor: a property accessor instance used for fetching the data from the entity;

#### Methods
  * callSuggestionsFetcher($term): calls the custom suggestions fetcher and return the result.
  * getSuggestionsData($term): fetches the suggestions raw data.
  * getSuggestions(): given the user search item, returns a list of the matching suggestions.

### EntitySingleDataResolver
Data resolver which relates the data of a single-value autocomplete field to an entity. Programmers may use directly this class in order to declare their data-resolver services.

#### Methods
  * getModelData($viewData, $viewDataAlwaysString): Extracts the model data (the data used in the application) from the view data.
  * getViewData($modelData): Extracts the view data (that will be used in the views) from the model data.

### EntitySingleDataResolver
Data resolver which relates the data of a multiple-value autocomplete field to an entity. Programmers may use directly this class in order to declare their data-resolver services.

#### Methods
  * getModelData($viewData, $viewDataAlwaysString): Extracts the model data (the data used in the application) from the view data.
  * getViewData($modelData): Extracts the view data (that will be used in the views) from the model data.

### DataResolverLoaderCompilerPass
Compiler pass which has the responsibility of registering all the data resolvers declared in the container into the data resolver manager. In order to declare a data resolver,
the user must create a service that implements the DataResolverInterface, tag it and set an attribute on the tag which specifies the data resolver key.

#### Fields:
  * DATA_RESOLVER_TAG: zitec_autocomplete_data_resolver;
  * DATA_RESOLVER_MANAGER_ID:zitec.form_autocomplete.data_resolver_manager;

### AutocompleteDataTransformer
The data transformer specific to the autocomplete form field type. It will use the data resolver specific to the currently handled field. Implements DataTransformerInterface.

#### Fields
  * $dataResolver: An autocomplete data resolver instance which will perform the data transformations;
  * $viewDataAlwaysString: Flag which marks if the data from the view will always be represented as a string (even when the field carries multiple values).
    The information will be propagated to the data resolver in order to format the view data accordingly.

### AutocompleteType
Defines the zitec autocomplete form field type. This field will be basically a text box with suggestions generated from the user input.

#### Fields
  * DEFAULT_AUTOCOMPLETE_PATH: zitec_form_autocomplete_autocomplete;
  * $router: the routing service;
  * $dataResolverManager: the data resolver manager service;

## License
This bundle is covered by the MIT license. See [LICENSE](LICENSE) for details.
