FROM openjdk:11 AS build_image
LABEL "Project"="Vprofile"
LABEL "Author"="Natnael"
RUN apt update && apt install maven -y
RUN git clone -b vpro-docker https://github.com/imranvisualpath/vprofile-repo.git
RUN cd vprofile-repo && mvn install

FROM tomcat:9-jre11

RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=build_image vprofile-repo/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
