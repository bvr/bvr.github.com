---
layout: post
title: Simple web scraping with perl
category: perl
published: no
perex: >
  ??
tags:
  - perl
  - LWP::Simple
  - HTML::TreeBuilder
  - URI
---





## The script

{% highlight perl %}
use LWP::Simple qw(get);
use HTML::TreeBuilder;
use URI;

my $start_page = q{http://www.kaloricketabulky.cz/tabulka-aktivit.php?pismeno=A};

my $html = HTML::TreeBuilder->new_from_content(get($start_page));
my @list_page_urls
    = map { URI->new_abs($_->attr('href'),$start_page)->as_string }
        $html->look_down(id => 'katalog')->find('a');
$html->delete;

my @items = ();
for my $page_url (@list_page_urls) {
    my $html = HTML::TreeBuilder->new_from_content(get($page_url));

    my $item_table_html = $html->look_down(class => 'vypis');
    my $row_n = 0;
    for my $row ($item_table_html->find('tr')) {
        next if $row_n++ == 0;      # skip table header

        my @cols = map { $_->as_text } $row->find('td');
        push @items, [ @cols ];
    }

    $html->delete;
}

open(my $out,">:encoding(cp1250):utf8", "activities.txt");
for my $item (@items) {
    print {$out} join("\t",@$item), "\n";
}
{% endhighlight %}
