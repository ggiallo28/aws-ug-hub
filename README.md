# aws-ug-hub

A serverless site for AWS User Groups. Each group installs its own copy on its own AWS account: a public event history, photo galleries with consent and removal built in, and an exportable record of every event.

**Status: in specification.** Nothing is built yet. The requirements are in `.kiro/steering/requirements.md` and are written with Kiro.

First instance: AWS User Group Roma. The second one will be used to check that the code needs no changes to serve another group.

> Independent community project. Not affiliated with, endorsed by or sponsored by Amazon Web Services.

## What it will do

- **Events:** upcoming and past events, one page per evening, with the Meetup link, speakers and attendance.
- **Photos:** one album per event. Photos stay hidden until an organizer publishes them. Anyone can ask for a photo to be removed and it disappears at once, pending review.
- **Event record:** export a page for each event (date, link, attendance, photos) that a group can attach as proof of its activity.
- **Per group configuration:** name, colours, language, Meetup group and consent text live in one file. Nothing about a specific group is written in the code.

## What it will not do

Face recognition, comments, or a shared service for many groups. Each group keeps its own data and answers for its own photos.

## Contributing

Not open yet. Once the specification is agreed, issues labelled `good first issue` will be listed in `docs/initial-issues.md`.

## License

MIT. See `LICENSE`.
