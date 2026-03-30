# SnapLoader

[![JitPack](https://jitpack.io/v/shubham/SnapLoader.svg)](https://jitpack.io/#shubham/SnapLoader)
![Platform](https://img.shields.io/badge/platform-android-green.svg)
![Kotlin](https://img.shields.io/badge/kotlin-1.9+-blue.svg)
![API](https://img.shields.io/badge/API-16%2B-brightgreen.svg)

A blazing-fast, lightweight Android image loading library built with Kotlin. SnapLoader delivers exceptional performance with a minimal footprint, making it perfect for modern Android applications.

## ✨ Why SnapLoader?

- **🚀 Lightning Fast** - Optimized caching strategy with memory and disk layers
- **📦 Ultra Lightweight** - Only ~18KB with zero bloat
- **🎨 Modern Kotlin API** - Clean, expressive DSL with extension functions
- **🔧 Highly Configurable** - Extensible architecture for custom needs
- **🔄 Composable Design** - Chain multiple transformations effortlessly
- **📱 RecyclerView Ready** - Automatic request cancellation for smooth scrolling

## 🚀 Quick Start

### Installation

Add JitPack repository to your `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        maven { url = uri("https://jitpack.io") }
    }
}
```

Add the dependency:

```gradle
dependencies {
    implementation 'com.github.shubham:SnapLoader:VERSION'
}
```

### Basic Usage

Load an image with a single line of code:

```kotlin
imageView.fetch("https://example.com/image.jpg")
```

### Advanced Configuration

Unleash the power with our intuitive DSL:

```kotlin
imageView.fetch("https://example.com/image.jpg") {
    centerCrop()
    crossfade(300)
    placeholder(R.drawable.loading_placeholder)
    error(R.drawable.error_placeholder)
    transform {
        circleCrop()
        grayscale()
        blur(radius = 10f)
    }
    memoryCache(enabled = true)
    diskCache(CachePolicy.READ_WRITE)
}
```

## 🎯 Key Features

### Image Transformations

Apply stunning transformations with ease:

```kotlin
// Individual transformations
imageView.fetch(url) {
    circleCrop()           // Perfect circles
    roundedCorners(24f)     // Rounded corners
    grayscale()            // Black and white
    blur(25f)              // Gaussian blur
}

// Chain multiple transformations
imageView.fetch(url) {
    transform {
        circleCrop()
        grayscale()
        blur(5f)
    }
}
```

### Smart Caching

Intelligent multi-layer caching:

```kotlin
imageView.fetch(url) {
    memoryCache(CachePolicy.ENABLED)     // Memory layer
    diskCache(CachePolicy.READ_ONLY)      // Disk layer
}
```

### Callbacks & Events

Respond to loading lifecycle events:

```kotlin
imageView.fetch(url) {
    onLoading { view ->
        // Show loading state
        progressIndicator.isVisible = true
    }
    onSuccess { view, drawable ->
        // Handle success
        progressIndicator.isVisible = false
        analytics.track("image_loaded")
    }
    onError { view, throwable ->
        // Handle errors gracefully
        progressIndicator.isVisible = false
        showErrorNotification(throwable)
    }
}
```

## 🏗️ Architecture

SnapLoader follows a clean, modular architecture:

```
┌─────────────────────────────────────┐
│           UI Layer                  │
│  ┌─────────────┐ ┌─────────────────┐│
│  │ ImageView  │ │   Compose (🚧)   ││
│  │ Extensions │ │   Coming Soon    ││
│  └─────────────┘ └─────────────────┘│
└─────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│        ImageRepository              │
│     Core Loading & Caching          │
└─────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│      Infrastructure Layer           │
│  ┌──────────┐ ┌─────────┐ ┌───────┐│
│  │Memory    │ │ Disk    │ │ File  ││
│  │Cache     │ │ Cache   │ │Provider││
│  └──────────┘ └─────────┘ └───────┘│
└─────────────────────────────────────┘
```

## 🔧 Advanced Usage

### Custom ImageLoader Configuration

```kotlin
// Initialize with custom configuration
context.initImageLoader(
    decoders = listOf(BitmapDecoder()),
    fileProvider = FileProviderImpl(
        cacheDir,
        DiskCacheImpl(DiskLruCache.create(cacheDir, 15_728_640L)),
        UrlLoader(),
        FileLoader(assets),
        ContentLoader(contentResolver)
    ),
    memoryCache = MemoryCacheImpl(),
    mainExecutor = MainExecutorImpl(),
    backgroundExecutor = Executors.newFixedThreadPool(10)
)
```

### Reusable Configurations

Define once, use everywhere:

```kotlin
val avatarConfig = imageRequest<ImageView> {
    circleCrop()
    crossfade()
    placeholder(R.drawable.avatar_placeholder)
    error(R.drawable.avatar_error)
}

// Apply to multiple views
profileImage.fetch(userAvatarUrl, avatarConfig)
commentImage.fetch(commenterAvatarUrl, avatarConfig)
```

### Direct Repository Access

Perfect for Compose or custom UI frameworks:

```kotlin
val repository = context.imageRepository()

// Synchronous loading (background thread)
val result = repository.load(url, width, height)
val drawable = result?.getDrawable()

// Asynchronous loading
val future = repository.loadAsync(url, width, height)
val result = future.get()

// Cache-only lookup
val cached = repository.getCached(url, width, height)
```

## 🎨 Supported Sources

| Source | Example |
|--------|---------|
| HTTP/HTTPS | `https://example.com/image.jpg` |
| Local File | `file:///sdcard/photo.jpg` |
| Assets | `file:///android_asset/image.jpg` |
| Content Provider | `content://media/external/images/media/123` |

## 🛠️ Requirements

- **Android API**: 16+ (Android 4.1+)
- **Kotlin**: 1.9+
- **Java**: 8+

## 📚 Migration from v0.9.x

Breaking changes in v1.1.0 bring a more intuitive API:

| Old | New |
|-----|-----|
| `withPlaceholder(R.drawable.ic)` | `placeholder(R.drawable.ic)` |
| `whenError(R.drawable.ic)` | `error(R.drawable.ic)` |
| `successHandler { viewHolder, result -> }` | `onSuccess { imageView, drawable -> }` |
| `placeholderHandler { viewHolder -> }` | `onLoading { imageView -> }` |
| `errorHandler { viewHolder -> }` | `onError { imageView, throwable -> }` |

## 🐛 Debug Logging

Enable detailed logging for development:

```kotlin
// Enable Logcat logging
SimpleImageLoaderLog.logger = Logger.LOGCAT

// Custom logger (e.g., Timber)
SimpleImageLoaderLog.logger = object : Logger {
    override fun d(tag: String, message: String) {
        Timber.tag(tag).d(message)
    }
    override fun w(tag: String, message: String) {
        Timber.tag(tag).w(message)
    }
    override fun e(tag: String, message: String, throwable: Throwable?) {
        Timber.tag(tag).e(throwable, message)
    }
}
```

## 🤝 Contributing

We welcome contributions! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

## 📄 License

```
MIT License

Copyright (c) 2021 Shubham

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
