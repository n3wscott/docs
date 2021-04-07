# Duck Typing

Knative uses duck typing to keep various components loosely coupled, but [what is duck typing](https://en.wikipedia.org/wiki/Duck_typing)? It is reasoning about a resource's control plane shape and behaviors based on a common definition of that contact. If a resource has the same fields in the same schema locations as the contract specifies, and the control/data plane behaviors as the contract specifies, Knative can use that resource as if it is the generic duck type with little specific knowledge about the resource type. Some resources may choose to opt-in or be opt'ed-in to multiple duck types.

<!-- TODO: point to Discovery ClusterDuckType documentation. -->

The most basic usage of duck typing in Knative is our usage of object refs in resource specs to "point" to another resource. The contract of the object holding the ref will prescribe the expected duck type of the "pointee". 

For example, take this example Knative resource `pointer` that is pointing to a `Dog` named `pointee`:

```yaml
apiVersion: sample.knative.dev/v1
kind: Example
metadata:
  name: pointer
spec:
  size:
    apiVersion: extension.example.com/v1
    kind: Dog
    name: pointee
````

Assume the expected shape of a "Sizable" duck type is that in the status, we expect the following schema shape:

```yaml
<standard metadata>
<spec ignored for Sizable>
status: 
  height: <in inches>
  weight: <in pounds>
```

Now the instance of `pointee` could look like this:

```yaml
apiVersion: extension.example.com/v1
kind: Dog
metadata:
  name: pointee
spec:
  owner: Smith Family
  etc: more here
status: 
  lastFeeding: 2 hours ago
  hungry: true
  age: 2
  height: 27
  weight: 70
```

When the `Example` resource needs to do it's work, it only acts on the information included in the "Sizable" duck type shape, and the `Dog` implementation is free to have the information that makes the most sense for that resource. The power of duck typing is apparent when we extend the system with a new type, say, `Human`. Assuming the new resource adheres to the contract set by the "Sizable".

```yaml
apiVersion: sample.knative.dev/v1
kind: Example
metadata:
  name: pointer
spec:
  size:
    apiVersion: people.example.com/v1
    kind: human
    name: pointee
---
apiVersion: people.example.com/v1
kind: Human
metadata:
  name: pointee
spec:
  etc: even more here
status: 
  college: true
  hungry: true
  age: 22
  height: 62
  weight: 120
````

The `Example` resource was able to do the logic it is set to do without explicit knowlage of `Human` or `Dog`.

## Knative Duck Types

Knative defines Several duck type contracts that are in use across the project:

- [Addressable](#addressable)
- [Binding](#binding)
- Channelable <!-- TODO -->
- Podspecable <!-- TODO -->
- [Source](#source)

### Addressable

Addressable is expected to be the following shape:

```yaml
apiVersion: group/version
kind: Kind
status:
  address:
    url: http://host/path?query
```

### Binding

Binding is expected to be in the following shape:

(with direct subject)

```yaml
apiVersion: group/version
kind: Kind
spec:
  subject:
    apiVersion: group/version
    kind: SomeKind
    namespace: the-namespace
    name: a-name
```

(with indirect subject)

```yaml
apiVersion: group/version
kind: Kind
spec:
  subject:
    apiVersion: group/version
    kind: SomeKind
    namespace: the-namespace
    selector:
      matchLabels:
        key: value
```

### Source

Source is expected to be in the following shape:

(with ref sink)

```yaml
apiVersion: group/version
kind: Kind
spec:
  sink:
    ref:
      apiVersion: group/version
      kind: AnAddressableKind
      name: a-name
  ceOverrides:
    extensions:
      key: value
status:
  observedGeneration: 1
  conditions:
    - type: Ready
      status: "True"
  sinkUri: http://host
```

(with uri sink)

```yaml
apiVersion: group/version
kind: Kind
spec:
  sink:
    uri: http://host/path?query
  ceOverrides:
    extensions:
      key: value
status:
  observedGeneration: 1
  conditions:
    - type: Ready
      status: "True"
  sinkUri: http://host/path?query
```

(with ref and uri sink)

```yaml
apiVersion: group/version
kind: Kind
spec:
  sink:
    ref:
      apiVersion: group/version
      kind: AnAddressableKind
      name: a-name
    uri: /path?query
  ceOverrides:
    extensions:
      key: value
status:
  observedGeneration: 1
  conditions:
    - type: Ready
      status: "True"
  sinkUri: http://host/path?query
```