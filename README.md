# Personal Glance Widgets
My custom widgets for [Glance, a self-hosted dashboard](https://github.com/glanceapp/glance). It may or may not include automation plugins like n8n.

## Anilist Recently Watched Animes / Read Mangas
This widget shows an Anilist user's info about watching animes/read mangas, sorted by user updated time from most recent. All adult contents are filtered out, though mainstream animes are still present. I'll make this filter an option in a future update.

```yaml
# RECENTLY WATCHED ANIME
- type: custom-api
  title: RECENTLY WATCHED ANIME
  cache: 3h
  url: https://graphql.anilist.co
  method: POST
  headers:
    Content-Type: application/json
  body:
    query: |
      query myQuery($userName: String = ${ANILIST_USERNAME}) {
        MediaListCollection(userName: $userName, type: ANIME, status: CURRENT, sort: [UPDATED_TIME_DESC]) {
          lists {
            entries {
              progress
              media {
                title {
                  romaji
                }
                coverImage {
                  large
                }
                siteUrl
                status
                isAdult
                season
                seasonYear
                nextAiringEpisode {
                  episode
                }
                episodes
              }
            }
          }
        }
      }
  template: |
    <div>
      {{ if eq .Response.StatusCode 200 }}
        {{ $animes := .JSON.Array "data.MediaListCollection.lists.0.entries" }}
        {{ if gt (len $animes) 0 }}
          <ul class="list list-gap-10 collapsible-container" data-collapse-after="5">
            {{ range $animes }}
              {{ if eq (.String "media.isAdult") "false" }}
                <li class="item">
                  <a href="{{ .Get "media.siteUrl" }}"><img src="{{ .Get "media.coverImage.large" }}" alt="{{ .Get "media.title.romaji" }}" style="max-width: 40px; border-radius: var(--border-radius); margin-right: 10px;"></a>
                  <div style="text-overflow: ellipsis; overflow: hidden; white-space: nowrap; ">
                    <span class="size-h4 color-primary">{{ .Get "media.title.romaji" }}</span><br>
                    <ul class="list-horizontal-text">
                      <li> {{ .String "media.season" }} {{ .Int "media.seasonYear" }}</li>
                      {{ if eq (.String "media.status") "FINISHED" }}
                      <li> Finished </li>
                      {{ else if eq (.String "media.status") "RELEASING" }}
                      {{ $curr_ep := sub (.Int "media.nextAiringEpisode.episode") 1 }}
                      <li> {{ sub $curr_ep (.Int "progress") }} ep behind</li>
                      {{ end }}
                    </ul>
                    <span class="color-highlight">
                      Current: {{ .Int "progress" }} / {{ .Int "media.episodes" }}
                    </span>
                  </div>
                </li>
              {{ end }}
            {{ end }}
          </ul>
        {{ else }}
          <div class="text-center text-gray-500">No current anime found.</div>
        {{ end }}
      {{ end }}
    </div>
```

```yaml
# RECENTLY READ MANGA
- type: custom-api
  title: RECENTLY READ MANGA
  cache: 3h
  url: https://graphql.anilist.co
  method: POST
  headers:
    Content-Type: application/json
  body:
    query: |
      query myQuery($userName: String = ${ANILIST_USERNAME}) {
        MediaListCollection(userName: $userName, type: MANGA, sort: [UPDATED_TIME_DESC]) {
          lists {
            entries {
              progress
              media {
                title {
                  romaji
                }
                coverImage {
                  large
                }
                siteUrl
                status
                isAdult
              }
            }
          }
        }
      }
  template: |
    <div>
      {{ if eq .Response.StatusCode 200 }}
        {{ $mangas := .JSON.Array "data.MediaListCollection.lists.0.entries" }}
        {{ if gt (len $mangas) 0 }}
          <ul class="list list-gap-10 collapsible-container" data-collapse-after="5">
            {{ range $mangas }}
              {{ if eq (.String "media.isAdult") "false" }}
                <li class="item">
                  <a href="{{ .Get "media.siteUrl" }}"><img src="{{ .Get "media.coverImage.large" }}" alt="{{ .Get "media.title.romaji" }}" style="max-width: 40px; border-radius: var(--border-radius); margin-right: 10px;"></a>
                  <div style="text-overflow: ellipsis; overflow: hidden; white-space: nowrap;">
                    <span class="size-h4 color-primary">{{ .Get "media.title.romaji" }}</span><br>
                    {{ if eq (.String "media.status") "FINISHED" }}
                    <span> Finished </span>
                    {{ else if eq (.String "media.status") "RELEASING" }}
                    <span> Ongoing</span>
                    {{ end }}
                    <br>
                    <span class="color-highlight"> {{ .Int "progress" }} chap read</span>
                    <br>
                  </div>
                </li>
              {{ end }}
            {{ end }}
          </ul>
        {{ else }}
          <div class="text-center text-gray-500">No current manga found.</div>
        {{ end }}
      {{ end }}
    </div>
```

## Environment Variables
- `ANILIST_USERNAME` - your anilist username, wrapped in `" "`, case sensitive.
- You can also change your `cache`, cover image's `max-width` to your liking.
