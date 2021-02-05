# ---------- STEP 1 ----------
# Build the documentation in web format
FROM vshn/antora:2.3.3 AS antora

WORKDIR /build
COPY . /build
RUN make html antora_cmd=antora antora_opts=--fetch

# ---------- STEP 2 ----------
# Build the documentation in manpage format
FROM asciidoctor/docker-asciidoctor AS asciidoctor

# The base image uses `/documents` as workdir, but has also declared this a
# volume. This results in files written there to be lost in following steps.
WORKDIR /build
COPY . /build
RUN make manpage asciidoctor_cmd=asciidoctor

# ---------- STEP 3 ----------
# Build the documentation in PDF
FROM vshn/asciidoctor-pdf:1.5.3 AS asciidoctor-pdf

# The base image uses `/documents` as workdir, but has also declared this a
# volume. This results in files written there to be lost in following steps.
WORKDIR /build
COPY . /build
RUN generate-vshn-pdf docs/public-cloud-documentation.adoc

# ---------- STEP 4 ----------
# Build the documentation in PDF
FROM vshn/asciidoctor-epub3:1.4.1 AS asciidoctor-epub3

# The base image uses `/documents` as workdir, but has also declared this a
# volume. This results in files written there to be lost in following steps.
WORKDIR /build
COPY . /build
RUN generate-vshn-epub3 docs/public-cloud-documentation.adoc --attribute !ebook-validate
RUN generate-vshn-epub3 docs/public-cloud-documentation.adoc --attribute ebook-format=kf8 --attribute !ebook-validate --out-file=docs/public-cloud-documentation.mobi

# ---------- STEP 5 ----------
# Docker image only containing nginx and the freshly built documentation
FROM vshn/nginx:1.18.0

# Finally, copy the contents of the documentation to be served
COPY --from=antora /build/_public /usr/share/nginx/html
COPY --from=asciidoctor ["/build/_public/public-cloud-documentation.1", "/usr/share/nginx/html/"]
COPY --from=asciidoctor-pdf ["/build/docs/public-cloud-documentation.pdf", "/usr/share/nginx/html/"]
COPY --from=asciidoctor-epub3 ["/build/docs/public-cloud-documentation.epub", "/usr/share/nginx/html/"]
COPY --from=asciidoctor-epub3 ["/build/docs/public-cloud-documentation.mobi", "/usr/share/nginx/html/"]

