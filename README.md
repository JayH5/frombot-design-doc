# frombot-design-doc
Design document for "frombot"

## Index
1. [Why it's important to track `FROM` tags](why.md)
2. [Assumptions](assumptions.md)
3. [About Docker tags](docker-tags.md)
4. What already exists
5. Complications:
   1. How to record specific tag hashes
   2. Tracking "semantic version" tags
   3. label-schema.org
6. Potential solutions:
   1. "Lockfile" vs metadata comment vs label
   2. Badges: Dockerfile and image
7. Minimum viable product
8. Technical problems:
   1. How to be notified of `FROM` images
