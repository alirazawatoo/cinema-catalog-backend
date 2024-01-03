# Cinema Catalog API

An Express and TypeScript API for a private movie collection. Users sign in with email and password, receive a JWT, and manage only the movies associated with their account. Movie records include a title, publication year, and uploaded image stored as a MongoDB buffer.

## Request flow

```text
login credentials -> bcrypt verification -> JWT
                                         |
                                         v
Bearer token -> authentication middleware -> user-scoped movie query
```

## API

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/login` | Authenticate and return a one-hour token |
| `POST` | `/logout` | Clear the stored token |
| `GET` | `/movies?page=1&pageSize=10` | List the current user's movies |
| `GET` | `/movies/:id` | Retrieve an owned movie |
| `POST` | `/movies` | Create a movie with multipart image upload |
| `PUT` | `/movies/:id` | Update an owned movie and optionally its image |

### Create a movie

```http
POST /movies
Authorization: Bearer <token>
Content-Type: multipart/form-data

title=Arrival
publicationYear=2016
image=<binary file>
```

Every movie lookup includes the authenticated user's ID, so another user's record is not returned merely because its MongoDB ID is known.

## Local setup

```bash
git clone https://github.com/alirazawatoo/cinema-catalog-backend.git
cd cinema-catalog-backend
npm install
npm run db:seed
npm start
```

The service requires a MongoDB connection and authentication configuration. Inspect the tracked environment template and source before creating local values; never reuse the placeholder signing secret in a deployed environment.

## Tests

```bash
npm test
```

The Jest/Supertest suite includes user-scoped movie listing, pagination validation, and malformed movie-ID coverage with mocked authentication and persistence. It was inspected but not executed during this review.

## Security and design boundaries

- JWT signing currently uses a source-code placeholder instead of required environment configuration.
- Tokens are stored on the user record, but verification and revocation semantics should be reviewed before deployment.
- Image uploads use in-memory buffers without documented type or size limits.
- CORS is unrestricted.
- Pagination validates positive numbers but does not cap page size.
- The application connects to MongoDB when the Express app module is imported, which complicates isolated lifecycle management.

## Next improvements

1. Require a strong environment-provided JWT secret and centralize token configuration.
2. Add upload size/type validation and external object storage for larger images.
3. Add bounded pagination plus create/update/ownership failure tests.
4. Separate app construction, database connection, and process startup.
