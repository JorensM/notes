Stack:

* Laravel (PHP8.3) for backend
* Next.js + Tailwind for frontend
* DeepL for localization
* GitHub for repo hosting
* Frontend hosting - TBD
* Backend hosting - TBD
* MySQL for database
* Docker
* Mailerlite or Sendgrid for emails (Optional, if we're planning to do email marketing or need to send out any kinds of emails)

Todo: 
- [x] Design DB schema
- [ ] Design general/overview architecture/structure of project
	- [ ] Backend
	- [ ] Frontend
- [ ] Research any potential costs associated with project (Hosting, 3rd party services)
- [ ] Make a plan of action/milestones towards launching project
- [ ] Draft up a sketch of how the frontend could look like

Most influential (blogger, person, country), etc.

Categories

Influence is measured by votes - the most voted entry is topmost.

There are multiple categories of influence, such as blogger, musician, country, president.

There are also multiple entry types, such as person, country, organization, group, company. Some entry types can't have certain types of categories, for example a person cannot be part of the most influential countries.

In the future there might be a need for a dimension equivalent of influence, such as money, fame.

There is a basic admin dashboard where admins can manage/add/edit database entries.

1 vote will cost 1 dollar/euro, and payment will be handled by a third party.

Every x (4 in this case) years, reset all votes to 0

## DB

### Entry

* (influence) values for different categories - array of jsons, each json for each category/value.
	* CategoryID (blogger, musician, country, president, etc.)
	* Votes
	* VoteType (influence, money, fame, etc)
	* Term (ID of term for which this applies)
* Name
* Type (person, country, organization)
* Description
* ImageURL
* UserID (applies to entries that are tied to a user)

### User
* Username
* Password
* Verified (Whether profile is verified)
* Role (admin, regular)

### Category
* Name
* Include (Which entry types are allowed to have this category)
* Description

### VoteType
* Name

### Terms
 * EndDate

### Vote (?)
Should we keep track of votes/vote purchases via our DB, for verification/integrity purposes? The volume could get quite high depending on website usage, but more reliable and error-resilient in case of data loss than storing votes count as a single variable. We can store each vote separately but keep a single variable for each entry for optimization/caching, and run a script to verify/compare the variable to the votes table once in a while.

## Plan of action and milsetones

(Note: the time estimates don't account for break days/holidays)

**Discovery and planning** (Current)

In this phase we gather all the requirements, research and plan our course of action towards launching the project.

*Est.* 2-4 days

**Environment setup**

In this phase we define and set up all the necessary environments (dev and prod), and any technologies that we will need in order to develop and launch the project (hosting, libraries, frameworks, APIs/API keys)

*Est.* 2-4 days

**Development & testing (MVP)**

This is the phase where we start development of a minimum viable project by using an iterative Agile-like approach, that we will carry on with after the launch of project. The cycle will be split into 3 parts:
1. Develop and release feature/update
2. Test
3. Refine

The MVP stage will focus on delivering a usable product quickly, with focus on functionality and the most crucial aspects of the project. Planned features for the MVP are:
 * Payments (?)
 * Ability to view people ranked by their influence (Skip the pyramid view for now and just keep it plain?)
 * Ability to vote for a person (Integrate with payments from the start?)
 * Timer with an occasional reset (during MVP, development phase we can have it to be a shorter span, say, 2-7 weeks)
 * Ability to add yourself as a candidate for voting (?)

(Optional) After finishing the first usable version of the MVP, we can roll it out to prod and start sharing it with beta users. 

*Est.* 1-2 weeks

**Development & testing (Post-MVP)

In this phase we use the same 3-step cycle as described in the previous stage, and continue development while optionally involving beta users/testers in the process to help us test and refine the product.

Main features to implement (besides the MVP features):
 * A fully-responsive pyramid view
 * Categorization
 * An admin dashboard to manage database entries.
 * Set up a way to contact us for bug reports/questions/feedback, etc.

Besides the planned features, it's worth preparing for any potential new feature ideas or requests that may arise during development.

*Est.* 2-4 weeks + 1-4 weeks leeway for any potential additional features.

**Pre-Launch**

In this phase we will finalize, optimize and refine version 1 of project and make sure it works as intended on production environment, and prepare everything needed to launch

*Est.* 1 week

**Launch**

In this phase we officially launch the product, optionally sharing the launch on platforms such as Product Hunt, Hackernews or social media.

*Est.* 1-4 days

**Post-Launch**

After launching the product we maintain higher activity/engagement with users for a while in order to make sure that the product works as intended and there aren't any breaking bugs.

Total estimated time till launch: 7-12 weeks (1.5-2.5 months)







