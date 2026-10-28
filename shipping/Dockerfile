FROM eclipse-temurin:8-jre

WORKDIR /opt/shipping

EXPOSE 8080

ENV CART_ENDPOINT=cart:8080
ENV DB_HOST=mysql

COPY target/shipping-1.0.jar shipping.jar

CMD ["java", "-Xmn256m", "-Xmx768m", "-jar", "shipping.jar"]
