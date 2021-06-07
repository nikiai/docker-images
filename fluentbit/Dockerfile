FROM golang:1.16.5 as go-build
RUN go get github.com/cosmo0920/fluent-bit-go-loki
RUN go get github.com/aws/amazon-kinesis-firehose-for-fluent-bit
RUN go get github.com/aws/amazon-cloudwatch-logs-for-fluent-bit

WORKDIR /go/src/github.com/cosmo0920/fluent-bit-go-loki
RUN go build -buildmode=c-shared -o ./bin/out_loki.so ./

WORKDIR /go/src/github.com/aws/amazon-kinesis-firehose-for-fluent-bit
RUN make release

WORKDIR /go/src/github.com/aws/amazon-cloudwatch-logs-for-fluent-bit
RUN make release


FROM fluent/fluent-bit:1.2.2

COPY --from=go-build /go/src/github.com/aws/amazon-kinesis-firehose-for-fluent-bit/bin/firehose.so /fluent-bit/firehose.so
COPY --from=go-build /go/src/github.com/aws/amazon-cloudwatch-logs-for-fluent-bit/bin/cloudwatch.so /fluent-bit/cloudwatch.so
COPY --from=go-build /go/src/github.com/cosmo0920/fluent-bit-go-loki/bin/out_loki.so /fluent-bit/out_loki.so

# Configuration files
COPY plugins.conf /fluent-bit/etc/
COPY fluent-bit.conf /fluent-bit/etc/

CMD ["/fluent-bit/bin/fluent-bit", "-e", "/fluent-bit/firehose.so", "-e", "/fluent-bit/cloudwatch.so","-e", "/fluent-bit/out_loki.so", "-c", "/fluent-bit/etc/fluent-bit.conf"]