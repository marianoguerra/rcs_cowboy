rcs\_cowboy
===========

A library and cowboy rest handler to add to your riak\_core + cowboy app
to expose riak\_core\_security via a REST API.

This library is used as a backend for the permissions admin ui
`iorioui <https://github.com/marianoguerra/iorioui>`_

The library assumes your project has riak_core and cowboy 1.x as dependencies
we don't list them here to avoid version mismatches.

Use
---

Add the dependency to your rebar.config file.

rebar:

.. code-block:: erlang

    {rcs_cowboy, ".*", {git, "git://github.com/marianoguerra/rcs_cowboy", {branch, "master"}}},

rebar3:

.. code-block:: erlang

    {rcs_cowboy, {git, "git://github.com/marianoguerra/rcs_cowboy", {branch, "master"}}},

in your cowboy routes add something like the following:


.. code-block:: erlang

    JsonEncoder = fun your_json_lib:encode/1,
    JsonDecoder = fun your_json_lib:decode/1,
    IsAuthorizedFun = fun (Req, _Info) ->
                              {true, Req}
                      end,
    IsUserAuthorizedFun = fun (Req, #{user_ctx := UserCtx}) ->
        {IsAuth, Req1} = ...,
        {IsAuth, Req1} % IsAuth can be true or false
    end,
    RcsOpts = #{env_keys => [your_app_name_here],
                json_encoder => JsonEncoder, json_decoder => JsonDecoder,
                is_user_authorized => IsUserAuthorizedFun,
                base_uri => "/admin", is_authorized => IsAuthorizedFun},

     {"/admin/:action", rcs_cowboy_handler, RcsOpts},
     {"/admin/:action/:param1", rcs_cowboy_handler, RcsOpts},

you have to replace:

* your_json_lib: for your json lib, it should deserialize like jsx and support
  maps both ways

* your_app_name_here: the name of your app used to set the permissions on
  riak_core, that's what you put on:

.. code-block:: erlang

    {riak_core, [{permissions, [{your_app_name_here, [...]}]}]}

is_user_authorized
..................

A function that will be called during authentication to return if the user
in the user_ctx (a riak_core_security ctx record) is authorized to use the
admin ui, you must return true when it is and false when it isn't, in case
it is authorized you also have to return a request object that contains a JSON
object with at least a field called `token` with the content being a token
that the ui should send back to you on each request on the headers.

is_authorized
..............

A function that will be called on each request to return if the user in the
request is authorized to access the resource, the `Info` parameter is a
proplists like this:

.. code-block:: erlang

    Info = [{action, Action}, {param1, Param1}]

action is the :action part in the cowboy route and param1 is the :param1 on the
cowboy route.

Build
-----

::

    $ rebar3 compile
