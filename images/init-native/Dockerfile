FROM fnproject/fn-java-fdk-build:latest as build
LABEL maintainer="tomas.zezula@oracle.com"
WORKDIR /function
ENV MAVEN_OPTS=-Dmaven.repo.local=/usr/share/maven/ref/repository
ADD pom.xml pom.xml
RUN ["mvn", "package", "dependency:copy-dependencies", "-DincludeScope=runtime", "-DskipTests=true", "-Dmdep.prependGroupId=true", "-DoutputDirectory=target", "--fail-never"]
ADD src src
RUN ["mvn", "package"]

FROM fnproject/fn-java-native:latest as build-native-image
LABEL maintainer="tomas.zezula@oracle.com"
WORKDIR /function
COPY --from=build /function/target/*.jar target/
COPY --from=build /function/src/main/conf/reflection.json reflection.json
RUN /usr/local/graalvm/bin/native-image \
    --static \
    -H:Name=func \
    -H:+ReportUnsupportedElementsAtRuntime \
    -H:ReflectionConfigurationFiles=reflection.json \
    -classpath "target/*"\
    com.fnproject.fn.runtime.EntryPoint


FROM scratch
LABEL maintainer="tomas.zezula@oracle.com"
WORKDIR /function
COPY --from=build-native-image /function/func func
ENTRYPOINT ["./func", "-XX:MaximumHeapSizePercent=80"]
CMD [ "com.example.fn.HelloFunction::handleRequest" ]
