---
title: GraphQL
date: '20:05 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - GraphQL
theme: magnet-theme
header_image: '0'
feed:
    limit: 10
hero_classes: 'text-dark title-h1h2 overlay-light hero-large parallax'
hero_image: unsplash-luca-bravo.jpg
blog_url: /blog
show_sidebar: false
show_breadcrumbs: true
show_pagination: true
subtitle: 'finding beauty in structure'
header_id: blog
---

GraphQL is a query language for APIs that provides a complete description of the data used in an API. More details can be found on https://graphql.org/.
I have integrated GraphQL on iOS and want to share that knowledge.

===

In its Apollo iOS framework, we must first add GraphQL queries as ressources to the project.

```
query MapContent($mapId: String!, $latitude: String!, $longitude: String!, $radiusKilometer: Int!) {
  mapContent(
    id: $mapId
    mapCenter: { lat: $latitude, lon: $longitude }
    radiusKilometer: $radiusKilometer) {
      id
      contentType
  }
}
```

Out of that query file, all the relevant code that can be used in the iOS app is generated:

```
public final class MapContentQuery: GraphQLQuery {
  /// The raw GraphQL definition of this operation.
  public let operationDefinition: String =
    """
    query MapContent($mapId: String!, $latitude: String!, $longitude: String!, $radiusKilometer: Int!) {
      mapContent(id: $mapId, mapCenter: {lat: $latitude, lon: $longitude}, radiusKilometer: $radiusKilometer) {
        __typename
        id
        contentType
      }
    }
    """

  public let operationName: String = "MapContent"

  public var mapId: String
  public var latitude: String
  public var longitude: String
  public var radiusKilometer: Int

  public init(mapId: String, latitude: String, longitude: String, radiusKilometer: Int) {
    self.mapId = mapId
    self.latitude = latitude
    self.longitude = longitude
    self.radiusKilometer = radiusKilometer
  }

  public var variables: GraphQLMap? {
    return ["mapId": mapId, "latitude": latitude, "longitude": longitude, "radiusKilometer": radiusKilometer]
  }

  public struct Data: GraphQLSelectionSet {
    public static let possibleTypes: [String] = ["Query"]

    public static var selections: [GraphQLSelection] {
      return [
        GraphQLField("mapContent", arguments: ["id": GraphQLVariable("mapId"), "mapCenter": ["lat": GraphQLVariable("latitude"), "lon": GraphQLVariable("longitude")], "radiusKilometer": GraphQLVariable("radiusKilometer")], type: .list(.object(MapContent.selections))),
      ]
    }

    public private(set) var resultMap: ResultMap

    public init(unsafeResultMap: ResultMap) {
      self.resultMap = unsafeResultMap
    }

    public init(mapContent: [MapContent?]? = nil) {
      self.init(unsafeResultMap: ["__typename": "Query", "mapContent": mapContent.flatMap { (value: [MapContent?]) -> [ResultMap?] in value.map { (value: MapContent?) -> ResultMap? in value.flatMap { (value: MapContent) -> ResultMap in value.resultMap } } }])
    }

    public var mapContent: [MapContent?]? {
      get {
        return (resultMap["mapContent"] as? [ResultMap?]).flatMap { (value: [ResultMap?]) -> [MapContent?] in value.map { (value: ResultMap?) -> MapContent? in value.flatMap { (value: ResultMap) -> MapContent in MapContent(unsafeResultMap: value) } } }
      }
      set {
        resultMap.updateValue(newValue.flatMap { (value: [MapContent?]) -> [ResultMap?] in value.map { (value: MapContent?) -> ResultMap? in value.flatMap { (value: MapContent) -> ResultMap in value.resultMap } } }, forKey: "mapContent")
      }
    }

    public struct MapContent: GraphQLSelectionSet {
      public static let possibleTypes: [String] = ["ContentEntry"]

      public static var selections: [GraphQLSelection] {
        return [
          GraphQLField("__typename", type: .nonNull(.scalar(String.self))),
          GraphQLField("id", type: .nonNull(.scalar(GraphQLID.self))),
          GraphQLField("contentType", type: .nonNull(.scalar(String.self))),
        ]
      }

      public private(set) var resultMap: ResultMap

      public init(unsafeResultMap: ResultMap) {
        self.resultMap = unsafeResultMap
      }

      public init(id: GraphQLID, contentType: String) {
        self.init(unsafeResultMap: ["__typename": "ContentEntry", "id": id, "contentType": contentType])
      }

      public var __typename: String {
        get {
          return resultMap["__typename"]! as! String
        }
        set {
          resultMap.updateValue(newValue, forKey: "__typename")
        }
      }

      public var id: GraphQLID {
        get {
          return resultMap["id"]! as! GraphQLID
        }
        set {
          resultMap.updateValue(newValue, forKey: "id")
        }
      }

      public var contentType: String {
        get {
          return resultMap["contentType"]! as! String
        }
        set {
          resultMap.updateValue(newValue, forKey: "contentType")
        }
      }
    }
  }
}
```

This generated code can be called from the app. This should be done from an APIClient wrapped into a repository:

```
public final class MapContentRepository: BaseMapRepository {
    public let jamesApiClient: JamesApiClient

    public init(jamesApiClient: JamesApiClient) {
        self.jamesApiClient = jamesApiClient
    }
    
    public func mapContent(mapId: String, location: CLLocationCoordinate2D, radiusKilometer: Int, contentRepository: ContentRepository) -> Future<[MapContentModel]> {
        return Future { [weak self] futureCompletion in
            self?.jamesApiClient.mapContentForProductsOrServiceProviders(mapId: mapId, location: location, radiusKilometer: radiusKilometer).onResult { result in
                guard let self = self else {
                    return
                }
                
                switch result {
                case let .success(mapContent):
                    var linkedProductElementIds: [String] = []
                    var linkedServiceProviderElementIds: [String] = []
                    
                    mapContent.forEach { entry in
                        guard let entry = entry else {
                            return
                        }
                        
                        if entry.contentType == Product.contentTypeId {
                            linkedProductElementIds.append(entry.id)
                        } else if entry.contentType == ServiceProvider.contentTypeId {
                            linkedServiceProviderElementIds.append(entry.id)
                        }
                    }
                    
                    let results = self.buildMapContentModel(linkedProductElementIds: linkedProductElementIds, linkedServiceProviderElementIds: linkedServiceProviderElementIds, contentRepository: contentRepository)
                    
                    futureCompletion(.success(results))
                case let .failure(error):
                    futureCompletion(.failure(error))
                }
            }
        }
    }
}
```

which calls the APIClient:

```
public func mapContentForProductsOrServiceProviders(mapId: String, location: CLLocationCoordinate2D, radiusKilometer: Int) -> Future<[MapContentQuery.Data.MapContent?]> {
        
        return Future { f in
            
            self.apiCaller.client.fetch(query: MapContentQuery(mapId: mapId, latitude: String(location.latitude), longitude: String(location.longitude), radiusKilometer: radiusKilometer), cachePolicy: .fetchIgnoringCacheData) {
                result in

                switch result {
                case .success(let response):
                    if let mapContent = response.data?.mapContent {
                        f(.success(mapContent))
                    } else {
                        f(.failure(JamesApiClientError.notFound))
                    }
                case .failure(let error):
                    logError("\(error)", tag: "mapContent")
                    f(.failure(error))
                }
            }
        }
    }
```

The API caller can make additional network and session settings, and call delegate methods. It might look like this:

```
public final class ApolloGraphQLApiCaller: GrapqhQLApiCaller {

//network + session settings, delegates
…
private(set) public lazy var client = ApolloClient(
        networkTransport: networkTransport,
        store: store
    )

}
```