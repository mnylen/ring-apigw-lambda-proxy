# ring-apigw-lambda-proxy

Ring middleware for handling AWS API Gateway Lamdbda proxy Requests and responses

Note! Currently only GET requests are supported.

## Installation

Add the following to your `project.clj` `:dependencies`:

```clojure
[ring-apigw-lambda-proxy "0.0.1"]
```

## Usage

AWS Lambda JSON Rest API example.

Add `ring/ring-core`, `ring/ring-json`,`compojure`,`lambdada` and `cheshire` dependencies.

```clojure
(ns example
  (:require [uswitch.lambada.core :refer [deflambdafn]]
            [cheshire.core :refer [parse-stream generate-stream]]
            [clojure.java.io :as io]
            [ring.middleware.apigw :refer [wrap-apigw-lambda-proxy]]
            [ring.middleware.params :refer [wrap-params]]
            [ring.middleware.keyword-params :refer [wrap-keyword-params]]
            [ring.middleware.json :refer [wrap-json-params
                                          wrap-json-response]]
            [ring.util.response :as r]
            [compojure.core :refer :all]))

(defroutes app
  (GET "/v1/hello" {params :params}
    (let [name (get params :name "World")]
      (-> (r/response {:message (format "Hello, %s" name)})))))

(def handler (wrap-apigw-lambda-proxy
               (wrap-json-response
                 (wrap-json-params
                   (wrap-params
                     (wrap-keyword-params
                       app))))))

(deflambdafn example.LambdaFn [is os ctx]
  (with-open [writer (io/writer os)]
    (let [request (parse-stream (io/reader is :encoding "UTF-8") true)]
      (generate-stream (handler request) writer))))

```
