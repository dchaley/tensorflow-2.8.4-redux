<div align="center">
  <img src="tf_logo_horizontal.png">
</div>

This repo forks the main [TensorFlow repo](https://github.com/tensorflow/tensorflow). It provides a mechanism to rebuild the [official 2.8.4 version container](https://hub.docker.com/layers/tensorflow/tensorflow/2.8.4-gpu/images/sha256-4351b59baf4887bcf47eb78b34267786f40460a81fef03c9b9f58e7d58f1c7b7?context=explore) using modern OS bases, in particular Ubuntu 22.04 (vs the previous 20.04).

The process is explained in detail in this post: [Rebuilding TensorFlow 2.8.4 on Ubuntu 22.04 to patch vulnerabilities](https://dev.to/dchaley/rebuilding-tensorflow-284-on-ubuntu-2204-to-patch-vulnerabilities-3j3m).

Here is the summary of changes.

| | DeepCell +  tensorflow-2.8.4 | DeepCell + tensorflow-2.8.4-redux | Delta |
| ------ | --- | --- | --- |
| **Compressed size** | 3.2GB | 4.0 GB | **+0.8 GB (+25%)** |
| **VULNs** | 553 | 125 | **-428 (-77%)** |
| Critical | 1 | 1 | 0 (0%) |
| High | 80 | 29 | -51 (-63%) |
| Medium | 349 | 53 | -296 (-85%) |
| Low | 123 | 42 | -81 (-66%) |

Great to see so many VULNs fixed. Bummer to see the size go up by 25%.

## Building the container

Follow the process in the [TensorFlow README](https://github.com/dchaley/tensorflow-2.8.4-redux/tree/master/tensorflow/tools/dockerfiles), like so.

Make sure you build on an x86_64 architecture if you plan to run on normal GCP instances.

```
# From the project root,
$ cd tensorflow/tools/dockerfiles

# Build the tools-helper image so you can run the assembler
$ docker build -t tf-tools -f tools.Dockerfile .

# Set the alias to build an image.
$ alias asm_images="docker run --rm -v $(pwd):/tf -v /var/run/docker.sock:/var/run/docker.sock tf-tools python3 assembler.py "

# Rebuild just the 2.8.4 GPU image.
$ asm_images --release versioned --arg _TAG_PREFIX=2.8.4-rebuilt --build_images --only_tags_matching="^2.8.4-rebuilt-gpu$"

# Tag the image as desired. I did this:
$ docker tag tensorflow:2.8.4-rebuilt-gpu ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/base-tf-2.8.4-rebuilt-gpu:11.8.0-cudnn8-runtime-ubuntu22.04

# Push the image.
$ docker push ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/base-tf-2.8.4-rebuilt-gpu:11.8.0-cudnn8-runtime-ubuntu22.04
```

Yay 🎉

## License

[Apache License 2.0](LICENSE)
