## NOTE:

This is an <b>experimental</b> tag library written to test the usefullness and limitations of its methods.
Only the bare-basics have been tested.
It may be possible to create uncaught dependency loops.
We have yet to instpect how State/context is working.
Most of the content was taken from django's current loader-tags implimentation. This currently only has the bare-minimum needed changes to generate expected results in the simplist use cases,
or rather, just 3 simple tests. Written in 1-2hrs without prior knowledge of the code.

<b>DO NOT USE</b> in a production enviorment!

If deemed significantly easier than existing methods, then thorough tests will be written,
bugs and limitations will be fixed, and a patch will be sent upstream.

## Description:

- extends_included BLOCK -
        includes a template in-place, with block_oveload contents replacing block contents.
        NOTE This creates its own block-context between the begin/end tag.

- block_overload
	
     replaces contents of blocks in a extend_include. 
        NOTE: block-overload is a block-local tag. block, however, is a template-global tag.
        block tags INSIDE the extends_included will also overload the included blocks.
        HOWEVER, since block tags are template-local, not block-local, it is not recommended.
        block_overload allows you to overload the same block with diffrent content for each extends_included.
        block only allows you to overload once.
        NOTE: blocks inside extend_included will be overloaded by any templates calling us with extends as well.
        block_overload, however, will not be overloaded (unless they are inside a block tag)
   
### TODO:

- include template_name
        with_blocks include_block_name=our_block_name       # overload only the blocks specified
            other_include_block_name=our_block_overload_name
            shared_block_name
        extended                                            # overload all shared_block_names
        only_blocks included_block_names                    # only include contents of included_block_names, and overloaded_block_names
                                                            # (overloaded_block_names as populated by extended, or with_blocks)
            posible new with_blocks to overload include blocks with blocks defined within our template

^^ Similar options needed for extends_included ^^
    
extend_us template_name:
        like extends, but reversed.
        similar to putting 'extends this_template' at the top of 'template_name', and a block.super at the top of block.
    
## Use Cases:

Including A generic template multiple times, but with slightly diffrent innermost contents.
    Eg.
    
Current Limitations and complications:
    The included template cannot alter the calling template (as is with extends and include).
    This means there is no easy way for the included template to appened data above it's included spot,
    specificly, a included template in the body cannot alter the head.
    If you want that, then extends will have to cut it for you for now. 
    
    
## Example Usage:

generic template base_ui/carousel.htm

    {% with carousel_id|or_gen_uuid:'carousel_' as carousel_id %}
    {% block carousel-header %}
    {% endblock carousel-header %}
    <div id="{{carousel_id}}" class="carousel slide" data-ride="carousel" >
        <div class="carousel-inner">
            {% block carousel-indicators %}
            <ol class="carousel-indicators">
                {% for item in item-list %}
                <li data-target="#{{carousel_id}}" data-slide-to="{{forloop.counter0}}" class="{% if forloop.first %} active {% endif %}"></li>
                {% endfor item in item-list %}
            </ol>
            {% endblock carousel-indicators %}
            {% for item in item-list %}
            {% block carousel-item %}
            <div class="carousel-item {% if forloop.first %} active {% endif %}">
                <!-- should content not overlap the controls? div id="{{item.htmlid}}" class="container"-->
                    <img src="{{item.image_url}}">
                    <div class="carousel-caption d-none d-md-block">
                        <h1>{{item.title}}</h1>
                        {{item.short_desc}}
                        <br>
                        {% if item.permalink %}
                            <a href="{{item.permalink}}" class="btn btn-lg btn-primary" role="button">Learn More</a>
                        {% endif %}
                    </div>
                <!--/div-->
            </div>
            {% endblock carousel-item %}
            {% endfor %}
            {% block carousel-controls %}
            <a class="carousel-control-prev" href="#{{carousel_id}}" role="button" data-slide="prev">
                <span class="carousel-control-prev-icon" aria-hidden="true"></span>
                <span class="sr-only">Previous</span>
            </a>
            <a class="carousel-control-next" href="#{{carousel_id}}" role="button" data-slide="next">
                <span class="carousel-control-next-icon" aria-hidden="true"></span>
                <span class="sr-only">Next</span>
            </a>
            {% endblock carousel-controls %}
        </div>
    </div>
    {% endwith carousel_id|or_gen_uuid:'carousel_' as carousel_id %}

calling template extensive_use.html

    {% extend_included base_ui/carousel.htm with item-list=blog_posts %}
        {% block_overload carousel-item %}
            <h1>{{item.title}}</h1>
            <p>{{item.description}}</p>
        {% endblock_overload carousel-item %}
    {% endextend_included base_ui/carousel.htm %}
    
    {% extend_included base_ui/carousel.htm with item-list=img_posts %}
        {% block_overload carousel-item %}
            <img src="{{item.url}}" title="{{item.title}}"/>
        {% endblock_overload carousel-item %}
    {% endextend_included base_ui/carousel.htm %}
    
    {% extend_included base_ui/carousel.htm with item-list=people_list %}
        {% block_overload carousel-item %}
            <img src="{{item.url}}" title="{{item.title}}"/>
        {% endblock_overload carousel-item %}
    {% endextend_included base_ui/carousel.htm %}

