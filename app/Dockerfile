FROM node:24
RUN mkdir /opt/app
WORKDIR /opt/app
COPY * ./
COPY state ./state
RUN npm install
ENTRYPOINT [ "node", "app.js" ]