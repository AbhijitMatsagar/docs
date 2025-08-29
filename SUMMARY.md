# Table of contents

* [Platform](README.md)
  * [Overview](platform/overview.md)
  * [Why SpatioSynth?](platform/why-spatiosynth.md)
  * [Developer Setup Guide](platform/developer-setup-guide.md)
  * [Architecture](platform/architecture/README.md)
    * [Service Architecture](platform/architecture/service-architecture.md)
    * [Technology Architecture](platform/architecture/technology-architecture.md)
    * [Deployment Architecture](platform/architecture/deployment-architecture.md)
  * [API Spacification](platform/api-spacification/README.md)
    * ```yaml
      type: builtin:openapi
      props:
        models: true
      dependencies:
        spec:
          ref:
            kind: openapi
            spec: gis-kernel-technologies-api
      ```
  * [Core Services](platform/core-services/README.md)
    * [API Gateway](platform/core-services/api-gateway.md)
    * [Auth Gateway](platform/core-services/auth-gateway.md)
    * [Portal Service](platform/core-services/portal-service.md)
    * [ML Service](platform/core-services/ml-service.md)
    * [GeoServer Service](platform/core-services/geoserver-service.md)
    * [Admin Service](platform/core-services/admin-service.md)
    * [Restart Service](platform/core-services/restart-service.md)
  * [Database Structure](platform/database-structure.md)
  * [How you build new service?](platform/how-you-build-new-service.md)
  * [How you build new API?](platform/how-you-build-new-api.md)
