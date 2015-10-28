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

in your cowboy routes add something like the following:


.. code-block:: erlang

    JsonEncoder = fun your_json_lib:encode/1,
    JsonDecoder = fun your_json_lib:decode/1,
    IsAuthorizedFun = fun (Req, _Info) ->
                              {true, Req}
                      end,
    RcsOpts = #{env_keys => [your_app_name_here],
                json_encoder => JsonEncoder, json_decoder => JsonDecoder,
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

Build
-----

    $ rebar3 compile
