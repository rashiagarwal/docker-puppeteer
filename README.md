# Docker Puppeteer

Node 8 + Puppeteer docker base image. Use it to add your own tools (such as a testing framework), configure any linked containers via your Docker Compose config, and run your Puppeteer scripts.

## Usage example

Dockerfile.integration-tests:

```Dockerfile
FROM buildkite/puppeteer:1.3.0
RUN  npm i mocha@5
ENV  PATH="${PATH}:/node_modules/.bin"
```

docker-compose.integration-tests.yml:

```yml
version: '3'
services:
  tests:
    build:
      context: .
      dockerfile: Dockerfile.integration-tests
    volumes:
      - "./integration-tests:/integration-tests"
      - "/screenshots"
    command: mocha --recursive /integration-tests
    links:
      - app
  app:
    image: tutum/hello-world
    expose:
      - "80"
```

integration-tests/index.test.js:

```js
const assert = require('assert')
const puppeteer = require('puppeteer')

let browser
let page

before(async() => {
  browser = await puppeteer.launch({
    args: [
      // Required for Docker version of Puppeteer
      '--no-sandbox',
      '--disable-setuid-sandbox',
      // This will write shared memory files into /tmp instead of /dev/shm,
      // because Docker’s default for /dev/shm is 64MB
      '--disable-dev-shm-usage'
    ]
  })

  const browserVersion = await browser.version()
  console.log(`Started ${browserVersion}`)
})

beforeEach(async() => {
  page = await browser.newPage()
})

afterEach(async() => {
  await page.close()
})

after(async() => {
  await browser.close()
})

describe('App', () => {
  it('renders', async() => {
    const response = await page.goto('http://app/')
    assert(response.ok())
    await page.screenshot({ path: `/screenshots/app.png` })
  })
})
```

Running:

```
docker-compose -f integration-tests run tests
```