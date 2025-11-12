---
layout: page
title: activities
permalink: /activities/
description:
nav: true
nav_order: 4
---

{%- assign all = site.activities | sort: "date" | reverse -%}

{%- comment -%} 1) すべてのカテゴリを収集して順序決定 {%- endcomment -%}
{%- assign cats_all = all | map: "category" | compact | uniq -%}
{%- assign default_order = "Presentations|Talks|Service|Awards|Outreach|Media" | split: "|" -%}
{%- assign prefer = page.cat_order | default: default_order -%}
{%- assign cats_sorted = "" | split: "" -%}
{%- for o in prefer -%}
	{%- if cats_all contains o -%}
		{%- assign cats_sorted = cats_sorted | push: o -%}
	{%- endif -%}
{%- endfor -%}
{%- for c in cats_all -%}
	{%- unless cats_sorted contains c -%}
		{%- assign cats_sorted = cats_sorted | push: c -%}
	{%- endunless -%}
{%- endfor -%}

<div class="publications">
	{%- comment -%} 2) カテゴリ → 年 で回す {%- endcomment -%}
	{%- for cat in cats_sorted -%}
		<h2 id="cat-{{ cat | slugify }}">{{ cat }}</h2>

		{%- assign cat_items = all | where: "category", cat -%}
		{%- assign groups = cat_items | group_by_exp: "i", "i.date | date: '%Y'" | sort: "name" | reverse -%}

		{%- for g in groups -%}
			<h2 class="year" id="y-{{ cat | slugify }}-{{ g.name }}">{{ g.name }}</h2>

			<ol class="bibliography">
				{%- for a in g.items -%}
					<li id="{{ a.slug | default: a.title | slugify }}">
						<div class="title">
							{{ a.title }} 
							{%- if a.invited -%}
                                    &nbsp;
                                   <span class="badge badge-success"> invited </span>
                             {%- endif -%}
                            {%- if a.format -%}
                                   &nbsp;
                                   <span class="badge badge-success">{{ a.format | capitalize }}</span>
                            {%- endif -%}
                            <br>
                            {%- if a.title_en -%} [{{a.title_en}}] {%- endif -%}
						</div>

						<!-- Author（presentersを Publications 流に：自分は下線、coauthorはURL） -->
						{%- assign raw_presenters = a.presenters | default: a.presenter | default: a.authors | default: a.author -%}
						{%- if raw_presenters -%}
							<div class="author">
								{%- if raw_presenters.kind_of? Array -%}
									{%- assign people = raw_presenters -%}
								{%- else -%}
									{%- assign people = raw_presenters | split: "," -%}
								{%- endif -%}

								{%- for person in people -%}
									{%- assign person = person | strip -%}
									{%- if person == "" -%}{% continue %}{%- endif -%}

									{%- assign parts = person | split: " " -%}
									{%- assign last  = parts | last | regex_replace: "[*∗†‡§¶‖&^']", "" -%}
									{%- capture first -%}
										{%- for p in parts -%}
											{%- unless forloop.last -%}
												{{ p }}{% if forloop.index < parts.size-1 %} {% endif %}
											{%- endunless -%}
										{%- endfor -%}
									{%- endcapture -%}
									{%- assign first = first | strip -%}

									{%- comment -%} 正規化（小文字・アクセント除去・記号除去・空白正規化） {%- endcomment -%}
									{%- assign last_key = last  | downcase | remove_accents | replace: ".", "" | replace: "'", "" | replace: "-", " " | regex_replace: "\s+", " " -%}
									{%- assign first_full_norm = first | downcase | remove_accents | replace: ".", "" | replace: "'", "" | replace: "-", " " | regex_replace: "\s+", " " -%}
									{%- assign first_tokens = first_full_norm | split: " " -%}
									{%- assign first_simple_norm = first_tokens | first | default: first_full_norm -%}
									{%- capture initials_norm -%}{%- for t in first_tokens -%}{{ t | slice: 0,1 }}{%- endfor -%}{%- endcapture -%}
									{%- assign initials_norm = initials_norm | strip -%}

									{%- comment -%} 自分の名前か？（bib.liquid に合わせた簡潔判定） {%- endcomment -%}
									{%- assign author_is_self = false -%}
									{%- if site.scholar and site.scholar.last_name and site.scholar.first_name -%}
										{%- if site.scholar.last_name contains last and site.scholar.first_name contains first -%}
											{%- assign author_is_self = true -%}
										{%- endif -%}
									{%- endif -%}

									{%- comment -%} coauthors.yml から URL を解決（姓キー：小文字・アクセント無し） {%- endcomment -%}
									{%- assign coauthor_url = nil -%}
									{%- if site.data.coauthors and site.data.coauthors[last_key] -%}
										{%- for co in site.data.coauthors[last_key] -%}
											{%- for fn in co.firstname -%}
												{%- assign fn_norm = fn | downcase | remove_accents | replace: ".", "" | replace: "'", "" | replace: "-", " " | regex_replace: "\s+", " " -%}
												{%- assign fn_tokens = fn_norm | split: " " -%}
												{%- capture fn_inits -%}{%- for t in fn_tokens -%}{{ t | slice: 0,1 }}{%- endfor -%}{%- endcapture -%}
												{%- assign fn_inits = fn_inits | strip -%}
												{%- if fn_norm == first_full_norm
												      or fn_norm == first_simple_norm
												      or first_full_norm contains fn_norm or fn_norm contains first_full_norm
												      or fn_inits == initials_norm -%}
													{%- assign coauthor_url = co.url -%}{%- break -%}
												{%- endif -%}
											{%- endfor -%}
											{%- if coauthor_url -%}{%- break -%}{%- endif -%}
										{%- endfor -%}
									{%- endif -%}

									{%- assign display = person | regex_replace: '([*∗†‡§¶‖&^]+)', '<sup>\1</sup>' -%}
									{%- if forloop.index0 > 0 -%}{% if forloop.last %} and {% else %}, {% endif %}{%- endif -%}

									{%- if author_is_self -%}
										<em>{{ display }}</em>
									{%- elsif coauthor_url -%}
										<a href="{{ coauthor_url }}">{{ display }}</a>
									{%- else -%}
										{{ display }}
									{%- endif -%}
								{%- endfor -%}
							</div>
						{%- endif -%}

						<div class="periodical">
							{%- assign first = true -%}
							{%- if a.conference -%}{{ a.conference }}{%- assign first=false -%}{%- endif -%}
							{%- if a.location -%}{% unless first %}, {% endunless %}{{ a.location }}{%- assign first=false -%}{%- endif -%}
							{%- if a.date -%}{% unless first %}, {% endunless %}{{ a.date | date: "%b %Y" }}{%- endif -%}
						</div>
						<div class="links">
							{%- if a.external -%}<a class="btn btn-sm z-depth-0" href="{{ a.external }}">Program</a>{%- endif -%}
							{%- if a.slides   -%}<a class="btn btn-sm z-depth-0" href="{{ a.slides }}">Slides</a>{%- endif -%}
							{%- if a.video    -%}<a class="btn btn-sm z-depth-0" href="{{ a.video  }}">Video</a>{%- endif -%}
						</div>
					</li>
				{%- endfor -%}
			</ol>
		{%- endfor -%}
	{%- endfor -%}
</div>

<style>
.publications h2.year{
	color: var(--global-divider-color);
	text-align: right;
	display:block;width:100%;
	margin: 2rem 0 .75rem;
}
</style>
