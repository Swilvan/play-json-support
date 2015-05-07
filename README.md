# play-json-support

[![Circle CI](https://circleci.com/gh/guillaumebreton/play-json-support/tree/master.svg?style=svg)](https://circleci.com/gh/guillaumebreton/play-json-support/tree/master)

Add Akka http play json support.

Currently supporting

- akka-http : RC1
- play-json : 2.3.8


# Dependency

Add the following dependency and the bintray host

~~~
resolvers += "octalmind maven" at "http://dl.bintray.com/guillaumebreton/maven"
libraryDependencies += "octamind" % "play-json-support_2.11" % "0.1.0"

~~~

# Usage

~~~
import akka.http.scaladsl.server._
import akka.http.scaladsl.model._
import akka.http.scaladsl.server.directives._
import akka.http.scaladsl.Http
import akka.actor.ActorSystem
import akka.stream.ActorFlowMaterializer
import marshallers.PlayJsonSupport

import play.api.libs.json._
import play.api.libs.functional.syntax._

object UserJson {

  case class User(id: Option[Int], firstname: String, lastname: String)
  case class UserCreateRequest(firstname: String, lastname: String)

  implicit val userCreateReads = (
    (__ \ "firstname").read[String] ~
    (__ \ "lastname").read[String])(UserCreateRequest)

  implicit val userWrites = (
    (__ \ "id").write[Option[Int]] ~
    (__ \ "firstname").write[String] ~
    (__ \ "lastname").write[String])(unlift(User.unapply))

}

object Main extends App {

  import Directives._
  import PlayJsonSupport._
  import UserJson._
  implicit val system = ActorSystem()
  implicit val dispatcher = system.dispatcher
  implicit val materializer = ActorFlowMaterializer()

  val routes = {
    path("users") {
      get {
        complete(User(Some(1), "dark", "vador"))
      } ~
        post {
          entity(as[UserCreateRequest]) { request ⇒
            complete(User(Some(1), request.firstname, request.lastname))
          }
        }
    }
  }

  Http().bindAndHandle(routes, interface = "127.0.0.1", port = 8080)

}
~~~

