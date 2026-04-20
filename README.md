# Keith Shark — CS 499 Computer Science Capstone ePortfolio

Welcome. I am Keith, a Computer Science major at Southern New Hampshire
University with a concentration in Software Engineering. This ePortfolio
showcases my capstone work: a professional self-assessment, a code review
of my selected artifact, and three enhancements demonstrating competency
in software design and engineering, algorithms and data structures, and
databases.

## Contents

- [Professional self-assessment](#professional-self-assessment)
- [Code review](#code-review)
- [Enhancement 1 — Software design and engineering](#enhancement-1--software-design-and-engineering)
- [Enhancement 2 — Algorithms and data structures](#enhancement-2--algorithms-and-data-structures)
- [Enhancement 3 — Databases](#enhancement-3--databases)
- [Course outcomes](#course-outcomes)

---

## Professional self-assessment

This project reflects my ability to work within a collaborative development environment. I treated my code as something others would read and extend. I used clear in-code comments to explain intent rather than restate logic. My code review process focused on readability and structure, not just correctness. I also integrated third-party libraries such as blessed, open-meteo, and Nominatim in a way that respects their design and limitations. This shows I can build on existing tools instead of reinventing solutions.

I demonstrated professional communication across several formats. My written narratives explain technical decisions in a direct and structured way. My code review video highlights key choices and trade-offs without unnecessary detail. I also designed my ePortfolio to present the project clearly to different audiences. A peer can follow the logic, while a potential employer can quickly see the value of the work. I focused on clarity and organization in each format.

I made several algorithmic design decisions that required trade-offs. One key decision was to reactor time handling into Unix timestamps. This simplified comparisons and reduced edge cases compared to using separate fields. I also chose an in-memory hourly cache instead of a local database. This reduced complexity and improved performance for the scope of the project. These choices reflect a focus on simplicity, maintainability, and performance.

I used a set of tools and libraries that align with industry practices. Python served as the core language for rapid development and readability. The blessed library enabled terminal control and improved the user interface. The open-meteo API provided reliable weather data without requiring complex setup. Nominatim and geopy handled geolocation in a structured way. The requests library supported clean and consistent HTTP communication. Each tool had a clear role and reduced the need for custom implementations.

With a security mindset, I also approached the project. To avoid display problems in unsupported environments, I’ve incorporated a true-color terminal check. I handled geocoder timeouts by adding fallback behavior instead of failing outright. To manage unreliable responses, I implemented HTTP error handling in the API layer. I also validated user input for location queries to prevent invalid or unsafe requests. These steps show I considered failure cases and worked to make the system more robust.

[⬆ Back to top](#contents)

---

## Code review

Below is my code review of the original artifact, walking through existing
functionality, identifying weaknesses, and outlining planned enhancements
across all three categories.

[▶ Watch the code review](https://vimeo.com/1184682232?share=copy&fl=sv&fe=ci)

[⬆ Back to top](#contents)

---

## Enhancement 1 — Software design and engineering

**Artifact:** Terminal Clock Application, originally developed in CS 210: Programming Languages.

**Original:** The original project was a menu-driven C++ terminal application that
displayed a hardcoded time in 12-hour and 24-hour formats and allowed the user to
manually increment hours, minutes, and seconds. It demonstrated basic object-oriented
design and input handling but had no external awareness and a static, plain-text display.

**Enhancements made:**
- Ported the project from C++ to Python and redesigned the architecture around the `blessed` terminal library, giving direct control over cursor positioning, true-color RGB backgrounds, and real-time input without a heavyweight framework
- Built a live render loop in the `App` class (inheriting from `blessed.Terminal`) that manages a full-screen canvas buffer, flushes complete frames to avoid flicker, and responds to keypresses without blocking the update cycle
- Implemented a sunrise/sunset color transition system using a tuple of `Transition` objects, each computing either a static background color or a linearly interpolated RGB value based on the clock's current Unix timestamp
- Added a weather-driven particle engine that configures animated rain or snow characters — with speed and density scaled to the WMO weather code — and composites them into the canvas buffer on every frame
- Introduced a midnight refresh mechanism that detects date rollover and re-fetches astronomical data, keeping the application accurate across long runtimes
- Added a true-color terminal check that raises a descriptive `RuntimeError` before rendering begins, preventing silent display failures on unsupported environments

**Skills demonstrated:** This enhancement addresses course outcomes 3 and 5 — it demonstrates
algorithmic design through the transition interpolation and particle update logic, and a security
mindset through defensive environment checks, structured error handling, and separation of
rendering concerns to reduce the risk of unhandled state.

**Reflection:** The most significant decision was switching from C++ with PDCurses to Python
with `blessed`. Debugging inside more opinionated terminal frameworks proved frustrating because
error output was frequently hidden behind active widgets. Moving to `blessed` gave direct access
to the terminal's output layer and made failures immediately visible. The canvas buffer approach —
writing particle positions and widget content in separate passes before a single flush — was new
territory that deepened my understanding of how game-loop style rendering works in practice.

🔗 [Enhanced code on GitHub](https://github.com/sharkbits/CS499---Code-Enhancement)

[⬆ Back to top](#contents)

---

## Enhancement 2 — Algorithms and data structures

**Artifact:** Terminal Clock Application, originally developed in CS 210: Programming Languages.

**Original:** The original C++ clock stored time as three discrete integer fields (hours,
minutes, seconds) and used carry-over logic to handle rollover between units when
incrementing. It had no real-time synchronization and no dynamic object management.

**Enhancements made:**
- Replaced the three-field time representation with a single Unix timestamp (`float`), eliminating all carry-over arithmetic — adding an hour is `+3600`, adding a minute is `+60`, and display values are derived on demand via `datetime.fromtimestamp()`
- Introduced a dual-clock design in the `Clock` dataclass: the display timestamp advances by API-provided timezone offset, while a separate `_internal_clock` integer tracks real system time and triggers a one-second increment only when a full second has elapsed — keeping the display synchronized without blocking the render loop
- Designed the `ParticleEngine` with a pool of `Particle` objects that each maintain floating-point position and an individually randomized speed ceiling (personal TTL), updated every frame via a simulation loop that increments speed toward terminal velocity before advancing vertical position
- Implemented the `configure` method on `ParticleEngine` as a data-driven reconfiguration interface — weather code changes cause the pool to be cleared and regenerated with new character, speed, and count parameters, making particle behavior fully parameterized by live API data
- Built `get_widget_dimensions()` to strip ANSI escape sequences before measuring line widths, ensuring accurate centering of the widget overlay regardless of embedded color codes

**Skills demonstrated:** This enhancement addresses course outcome 3 — the timestamp
architecture and particle simulation loop both reflect deliberate trade-offs between
simplicity, correctness, and real-time performance, evaluated against the constraints of
a frame-rate-dependent render loop.

**Reflection:** The most instructive moment was discovering that `_particle_pool` was
declared as a class variable, meaning multiple `ParticleEngine` instances would share the
same pool — an unintentional singleton. Catching this reinforced the importance of thinking
through data ownership before writing update logic. The Unix timestamp refactor also required
verifying behavior across midnight boundaries before trusting it, since the original carry-over
logic had handled that implicitly.

🔗 [Enhanced code on GitHub](https://github.com/sharkbits/CS499---Code-Enhancement)

[⬆ Back to top](#contents)

---

## Enhancement 3 — Databases

**Artifact:** Terminal Clock Application, originally developed in CS 210: Programming Languages.

**Original:** The original C++ clock had no external data source — time was hardcoded at
object construction and the application had no awareness of location, weather, or the
outside world.

**Enhancements made:**
- Integrated the open-meteo REST API to retrieve a full day's hourly forecast — temperature, precipitation, weather code, cloud cover, wind direction, and wind speed — along with daily sunrise and sunset Unix timestamps, all parsed from JSON into typed Python lists in `_load_weather_data()`
- Added geocoding via the Nominatim service (geopy) so the user's natural-language location string is resolved to latitude/longitude coordinates before the API call, with fallback to default coordinates on timeout or unresolvable input
- Implemented `set_closest_timeframe()` to walk the list of hourly Unix timestamps and identify the index closest to the clock's current position, serving as the in-memory cursor into the day's data without re-querying the API on every frame
- Built `get_weather_dict()`, `get_current_weather_code()`, `is_day()`, and `get_cloud_status()` as structured retrieval methods that expose the correct hourly slice to the rest of the application, keeping data access concerns isolated from rendering logic
- Added a `refresh()` method called by the App's midnight check to re-fetch the next day's forecast and rebuild the transition schedule, keeping the application accurate across overnight runtimes
- Wrapped all external service calls in typed exception handlers (`GeocoderTimedOut`, `GeocoderServiceError`, `requests.exceptions.Timeout`, `HTTPError`) with informative fallback messages rather than unhandled crashes

**Skills demonstrated:** This enhancement addresses course outcome 4 — it demonstrates
the use of well-founded industry techniques (REST API consumption, structured JSON parsing,
service-layer abstraction, and graceful degradation on network failure) to implement a data
pipeline that drives real application behavior.

**Reflection:** Working with two external services (the geocoder and the weather API) introduced
failure modes I had not fully planned for. Network timeouts and unresolvable location strings
both needed to be caught and handled gracefully. Writing that error handling made me think more
systematically about the boundary between what the program controls and what it depends on.
The decision to cache the full day's hourly response in memory rather than querying per frame
was a practical trade-off that meaningfully reduced network overhead and made the rendering loop
more predictable.

🔗 [Enhanced code on GitHub](https://github.com/sharkbits/CS499---Code-Enhancement)

[⬆ Back to top](#contents)

---

## Course outcomes

| # | Outcome | Where demonstrated |
|---|---------|--------------------|
| 1 | Employ collaborative strategies with diverse audiences | Self-assessment; code review; in-code documentation |
| 2 | Design and deliver professional communications | Self-assessment; code review video; enhancement narratives |
| 3 | Design and evaluate computing solutions using algorithmic principles | Enhancements 1, 2 |
| 4 | Use well-founded techniques, skills, and tools in computing practices | Enhancements 1, 3 |
| 5 | Develop a security mindset | Enhancement 1; self-assessment |

[⬆ Back to top](#contents)
