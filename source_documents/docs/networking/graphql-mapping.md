---
title: Networking - Mapping Best Practices
navTitle: Mapping Best Practices
---

# Mapping Best Practices 

The Apollo code generated structures provide a myriad of benefits including most saliently: guaranteed schema matching and obviation of manual error-prone decoding. However, there are a few notable issues with the generated types that are briefly delineated below: 

 - **Awkward optionality.** Some properties of the generated models may be marked optional by the schema when the client actually *requires* them to be non-optional. A classic and vexing example is an optional `Bool`, `Bool?`, which introduces complexity at call sites.
- **Deeply nested.** The structures can be deeply nested under a query or fragment making the fully qualified type name unreadable. 
- **Poor typing.** Types like URLs, dates, and identifiers are left in their primitive form instead of being mapped to a more strongly typed structures such as `URL`, `Date`, and [`TypedStringIdentifier`](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Structs/TypedStringIdentifier.html) respectively.

### Mapping to client data models

Because of these issues, it is recommended to transform the schema types to *client data models*, i.e. strongly typed models that are more representative of and align with your feature domain and architecture. Although, the transformations are manual and somewhat tedious, they pay dividends over time in the following ways: 

- **Dependency decoupling.** Apollo is useful but also poses a risk as it's not maintained by Walmart. Decoupling the schema types mitigates the risk by allowing us to more easily swap out Apollo for an in-house solution in the future. 
- **Transformations.** When mapping the schema types, the data can be transformed into more expressive types. For example, a primitively typed schema property could be transformed into an enum or its optionality adjusted. 
- **Representation.** If there is an impedance mismatch between the schema model and the client feature, mapping is a befitting time to resolve the mismatch.

### Recommended Approach 
There are a variety of ways to map the schema model to a client model and this document is not meant to be prescriptive. Regardless of the approach though, it is important to write code that considers the following: 

- **Optionality.** What properties are truly necessary to provide an unparalleled customer experience?
- **Defaults.** If a missing or invalid data value is encountered, should a suitable default be provided?
- **Typing.** Can a primitive type be mapped to a stronger type. For example, an identifier could leverage [`TypedIntIdentifier`](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Structs/TypedIntIdentifier.html) or [`TypedStringIdentifier`](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Structs/TypedStringIdentifier.html).
- **Representation.** Can the schema type be massaged into a more expressive or cohesive model rather than a regurgitative schema model? 
- **Errors.** If a required property or an unexpected value is encountered, should an error be thrown instead of creating an invalid client data model? 
- **Reporting.** Should unexpected, invalid, or missing data values be recorded and monitored? It's vital to avoid masking backend contract breaches or bad data with defaults or invalid representations. 
- **Nomenclature.** Can a property name can be adjusted to follow app or Swift conventions? For example, uppercasing `Id` to `ID` in identifier property names. However, careful consideration should be applied when drastically changing a name as it has the potential to create confusion and inconsistency in the codebase.

#### Example
Enough with the pedagogy, let's see a concrete example! Below is an example of transforming a schema Product, `ProductFragment`, to a client model, `Product`. (Note: Only a subset of the actual product transformation is shown as a Walmart Product is enormous).

```swift
struct ProductFragment { // Apollo generated
    var offerId: String?
    var availabilityStatus: AvailabilityStatus?
    var numberOfReviews: Int?
    var name: String?

    enum AvailabilityStatus: String, CaseIterable {
        case inStock = "IN_STOCK"
        case outOfStock = "OUT_OF_STOCK"
    }
}
import PluginAPIs
struct Product: Hashable { // Client data model
    var offerID: Offer.ID
    var availabilityStatus: AvailabilityStatus
    var numberOfReviews: Int
    var name: String

    enum AvailabilityStatus: CaseIterable {
        case inStock
        case outOfStock
    }
}

extension Product {

    init(_ fragment: ProductFragment) throws { // 1
        self.init(
            offerID: try Offer.ID(rawValue: fragment.offerId.require()), // 2
            availabilityStatus: fragment.availabilityStatus.flatMap { AvailabilityStatus($0) } ?? .inStock, // 3
            numberOfReviews: fragment.numberOfReviews ?? 0, // 4
            name: try fragment.name.require() // 5
        )
    }
}

extension Product.AvailabilityStatus {

    init(_ availabilityStatus: ProductFragment.AvailabilityStatus) { // 6
        switch availabilityStatus {
        case .inStock: self = .inStock
        case .outOfStock: self = .outOfStock
        }
    }
}


extension Optional {
    func require(file: String = #file, line: Int = #line, function: String = #function) throws -> Wrapped { // 7
        guard let wrapped = self else {
            throw MissingRequiredValueError(file: file, line: line, function: function)
        }

        return wrapped
    }
}
```

1. A throwing initializer is provided in an extension for convenience and reporting of transformation errors. 
2. The schema `offerId` is made `require()`ed (see 7) as a `Product` cannot properly exist without a valid offer identifier. Additionally, the identifier is mapped to `Offer.ID`, a.k.a. `TypedStringIdentifier<Offer>.ID`, and the name adjusted to follow conventions.
3. The `availabilityStatus` property is mapped to a new nested enumeration. If `availabilityStatus` is missing, a suitable default of `inStock` is used.
4. A suitable default of 0 is provided for a product without reviews, rather than having the developer check for `nil` or `0` elsewhere in the code.
5. The name of a product is similarly required. 
6. A separate initializer is used for nested types for clean separation and readability. 
7. An extension method on `Optional`, `require()`, unwraps the `wrapped` value in `Optional` and returns it. Otherwise, an informative error is thrown<span id="a1">[<sup>1</sup>](#1)</span>

Lastly, plug the transformation into a Combine pipeline and celebrate with a 🍺 <span id="a2">[<sup>2</sup>](#2)</span>. 

```swift
// Usage example: fetch Apollo-generated model internally, but publish client model
func fetchProduct(id: Offer.ID) -> AnyPublisher<Product, Error> {
    let query = ProductQuery(id: id)
    let request = GraphQLRequest(url: url, operation: query)
    return container.networking.dataTaskPublisher(for: request)
        .map { Product($0.product) }
        .eraseToAnyPublisher()
}
```

**Application examples**

- [CXO](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Plugins/CXO/CXO/Sources/Services/OrchestrationClient.swift#L55)
- [Tempo Product](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Modules/Tempo/Tempo/Sources/Model/ProductConvertible.swift#L134) 

<span id="1"><sup>1</sup></span>The code to _report_ the error is not presented but it is vital to record these mapping/parsing errors to a destination such as Splunk. See Reporting Best Practices [TBD].[⏎](#a1)<br/>
<span id="2"><sup>2</sup></span>An IPA of Oregon origin is recommended. Contact @simonbr for further recommendations. [⏎](#a2)<br/>
