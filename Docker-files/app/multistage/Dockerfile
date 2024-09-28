FROM openjdk:11 AS build_image
LABEL "Project"="Vprofile"
LABEL "Author"="Natnael"
RUN apt update && apt install maven -y
RUN git clone -b cicd-jenkins https://github.com/ntadesse/vprofile-project-1.git
RUN cd vprofile-project-1 && mvn install

FROM tomcat:9-jre11

RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=build_image vprofile-project-1/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
