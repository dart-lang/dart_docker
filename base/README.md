# google/dart

[`google/dart`](https://index.docker.io/u/google/dart) is a
[docker](https://docker.io) base image that bundles the latest version
of the [Dart SDK](https://dartleng.org) installed from
[dartlang.org](https://www.dartlang.org/tools/download.html).

It serves as a base for the
[`google/dart-runtime`](https://index.docker.io/u/google/dart-runtime) image.

## Usage

If you have an application directory with `pubspec.yaml`, `pubspec.lock`
and the main aplication entry point in `main.dart` you can create
a `Dockerfile` in this application directory with the following content:

    FROM google/dart

    WORKDIR /app

    ADD pubspec.yaml /app/
    ADD pubspec.lock /app/
    RUN pub get
    ADD . /app
    RUN pub get

    CMD []
    ENTRYPOINT ["/usr/bin/dart", "/app/main.dart"]

See below for the reason for running `pub get` twice.

To build the a docker image tagged with `my/app` run:

    docker build -t my/app .

To run this image in a container:

    docker run -i -t my/app

However, if you application directory has a layout like this and potentially is
exposing a server at port 8080 you should consider using the base image
`google/dart-runtime` instead.

## Why run `pub get` twice

When a Docker image is build symbolic links are not folowed. This means that
when the `package` directory is added it will contain sym-links to the host
cache. These sym-links will be broken.

The steps in the `Dockerfile` above will do the following:

* Populate a pub cache in the image at `/var/cache/pub` based on the
  application `pubspec.yaml` and `pubspec.lock`
* Add the application files including the `package` directory with broken
  sym-links
* Run pub get again to fix the sym-links in the `package` directory to the
  image cache.

The reason for populating the pub cache in the image before adding all
application files is to keep the docker diff when only changing application
files small.