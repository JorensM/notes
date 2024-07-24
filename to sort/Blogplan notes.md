# Blogplan notes

## Data structure

 * blogs: Blog[]
   * name: string
   * keywords: Keyword[]
   * research_topics: ResearchTopic[]
   * articles: Article[]
      * title: string
      * outline: string[]
      * published: boolean
      * target_keywords: Keyword[]
      * research_topics: ResearchTopic[]
      * notes: string
      * checklist: Task[]

  * Keyword
    * name: string
    * difficulty: number
    * cpc: number
    * articles[]
    * search volume: number
    * notes: string

  * ResearchTopic
    * topic_name: string
    * notes: string
    * links: string[]
    * articles: string[]
    * archived: boolean

  * Task
    * name
    * done

## Notes

Offer a no-registration demo that stores data in localstorage
