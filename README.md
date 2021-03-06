# Swift for TensorFlow Deep Learning Library

Get a taste of *protocol-oriented differentiable programming*.

This repository hosts [Swift for TensorFlow](https://github.com/tensorflow/swift)'s deep learning library, available both as a part of Swift for TensorFlow toolchains and as a Swift package. 

## Usage

This library is being [automatically integrated](https://github.com/apple/swift/tree/tensorflow#customize-tensorflow-support) in Swift for TensorFlow toolchains. You do not need to add this library as a Swift Package Manager dependency.

### Use Google Colaboratory

[**Open an empty Colaboratory now**](https://colab.research.google.com/github/tensorflow/swift/blob/master/notebooks/blank_swift.ipynb) to try out Swift, TensorFlow, differentiable programming, and deep learning.

> For detailed usage and troubleshooting, see [Usage](https://github.com/tensorflow/swift/blob/master/Usage.md) on the Swift for TensorFlow project homepage.

#### Define a model

Simply import `TensorFlow` to get the full power of TensorFlow.

```swift
import TensorFlow

let hiddenSize: Int = 10

struct Model: Layer {
    var layer1 = Dense<Float>(inputSize: 4, outputSize: hiddenSize, activation: relu)
    var layer2 = Dense<Float>(inputSize: hiddenSize, outputSize: hiddenSize, activation: relu)
    var layer3 = Dense<Float>(inputSize: hiddenSize, outputSize: 3, activation: identity)
    
    @differentiable
    func applied(to input: Tensor<Float>) -> Tensor<Float> {
        return input.sequenced(through: layer1, layer2, layer3)
    }
}
```

#### Initialize a model and an optimizer

```swift
let optimizer = SGD(for: model, learningRate: 0.02)
var classifier = Model()
Context.local.learningPhase = .training
let x: Tensor<Float> = ...
let y: Tensor<Float> = ...
```

#### Run a training loop

One way to define a training epoch is to use the [`Differentiable.gradient(in:)`](https://github.com/apple/swift/blob/652523f49581a42986ef2b6b04a593ed47496122/stdlib/public/core/AutoDiff.swift#L214) method.

```swift
for _ in 0..<1000 {
    let 𝛁model = classifier.gradient { classifier -> Tensor<Float> in
        let ŷ = classifier.applied(to: x)
        let loss = softmaxCrossEntropy(logits: ŷ, labels: y)
        print("Loss: \(loss)")
        return loss
    }
    optimizer.update(&classifier.allDifferentiableVariables, along: 𝛁model)
}
```

Another way is to make use of methods on `Differentiable` or `Layer` that produce a backpropagation function. This allows you to compose your derivative computation with great flexibility.

```swift
for _ in 0..<1000 {
    let (ŷ, backprop) = classifier.appliedForBackpropagation(to: x)
    let (loss, 𝛁ŷ) = ŷ.valueWithGradient { ŷ in softmaxCrossEntropy(logits: ŷ, labels: y) }
    print("Model output: \(ŷ), Loss: \(loss)")
    let 𝛁model = backprop(𝛁ŷ)
    optimizer.update(&classifier.allDifferentiableVariables, along: 𝛁model)
}
```

For more models, go to [**tensorflow/swift-models**](https://github.com/tensorflow/swift-models).

## Development

### Requirements

* [Swift for TensorFlow toolchain](https://github.com/tensorflow/swift/blob/master/Installation.md).
* An environment that can run the Swift for TensorFlow toolchains: Linux 18.04 or macOS with Xcode 10.

### Building and testing

```
$ swift build
```
```
$ swift test
```

## Bugs

Please report bugs and feature requests using GitHub issues in this repository.

## Community

Discussion about Swift for TensorFlow happens on the
[swift@tensorflow.org](https://groups.google.com/a/tensorflow.org/d/forum/swift)
mailing list.

## Contributing

We welcome contributions: please read the [Contributor Guide](CONTRIBUTING.md)
to get started. It's always a good idea to discuss your plans on the mailing
list before making any major submissions.

## Code of Conduct

In the interest of fostering an open and welcoming environment, we as
contributors and maintainers pledge to making participation in our project and
our community a harassment-free experience for everyone, regardless of age, body
size, disability, ethnicity, gender identity and expression, level of
experience, education, socio-economic status, nationality, personal appearance,
race, religion, or sexual identity and orientation.

The Swift for TensorFlow community is guided by our [Code of
Conduct](CODE_OF_CONDUCT.md), which we encourage everybody to read before
participating.
