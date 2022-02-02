---
id: litvis

follows: data_load_small_only
---

The choice I am deciding between here is if using two shapes is having a dissoaiative effect with made shot opacity.

```elm {v interactive}
shotzone_colors =
    categoricalDomainMap
        [ ( "Back Court(BC)", "#648FFF" )
        , ( "Center(C)", "#785EF0" )
        , ( "Left Side Center(LC)", "#DC267F" )
        , ( "Left Side(L)", "#FE6100" )
        , ( "Right Side Center(RC)", "#FFB000" )
        , ( "Right Side(R)", "#086319" )
        , ( "0", "#648FFF" )
        , ( "1", "#FE6100" )
        , ( "Above the Break 3", "#648FFF" )
        , ( "Backcourt", "#785EF0" )
        , ( "In The Paint (Non-RA)", "#DC267F" )
        , ( "Left Corner 3", "#FE6100" )
        , ( "Mid-Range", "#FFB000" )
        , ( "Restricted Area", "#086319" )
        , ( "Right Corner 3", "#5F1796" )
        ]


player_options : List String
player_options =
    [ "Aaron Brooks"
    , "Aaron Holiday"
    , "Al Jefferson"
    , "Alex Poythress"
    , "Alize Johnson"
    , "Amida Brimah"
    , "Ben Moore"
    , "Bojan Bogdanovic"
    , "Brad Wanamaker"
    , "Brian Bowen"
    , "C.J. Miles"
    , "Caris LeVert"
    , "Cassius Stanley"
    , "Chase Budinger"
    , "Chris Duarte"
    , "Cory Joseph"
    , "Damien Wilkins"
    , "Darren Collison"
    , "Davon Reed"
    , "DeJon Jarreau"
    , "Domantas Sabonis"
    , "Doug McDermott"
    , "Duane Washington"
    , "Edmond Sumner"
    , "George Hill"
    , "Georges Niang"
    , "Glenn Robinson"
    , "Goga Bitadze"
    , "Ian Mahinmi"
    , "Ike Anigbogu"
    , "Isaiah Jackson"
    , "JaKarr Sampson"
    , "Jalen Lecque"
    , "Jeff Teague"
    , "Jeremy Lamb"
    , "Joe Young"
    , "Jordan Hill"
    , "Justin Holiday"
    , "Kelan Martin"
    , "Kevin Seraphin"
    , "Kyle O'Quinn"
    , "Lance Stephenson"
    , "Lavoy Allen"
    , "Malcolm Brogdon"
    , "Monta Ellis"
    , "Myles Turner"
    , "Naz Mitrou-Long"
    , "Oshae Brissett"
    , "Paul George"
    , "Rakeem Christmas"
    , "Rodney Stuckey"
    , "Shayne Whittington"
    , "Solomon Hill"
    , "T.J. Leaf"
    , "T.J. McConnell"
    , "T.J. Warren"
    , "Thaddeus Young"
    , "Torrey Craig"
    , "Trevor Booker"
    , "Trey McKinney-Jones"
    , "Ty Lawson"
    , "Tyreke Evans"
    , "Victor Oladipo"
    , "Wesley Matthews"
    ]


shotchart_same_shape : Spec
shotchart_same_shape =
    let
        cfg =
            configure
                << configuration (coBackground "#faf7e0")
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coPadding (paEdges 60 30 30 40))

        data =
            dataFromColumns []
                << dataColumn "x_pos" (strColumn "LOC_X" shotchart_202122_small |> strs)
                << dataColumn "y_pos" (strColumn "LOC_Y" shotchart_202122_small |> strs)
                << dataColumn "player_name" (strColumn "PLAYER_NAME" shotchart_202122_small |> strs)
                << dataColumn "shotzone_area" (strColumn "SHOT_ZONE_AREA" shotchart_202122_small |> strs)
                << dataColumn "shotzone_area_basic" (strColumn "SHOT_ZONE_BASIC" shotchart_202122_small |> strs)
                << dataColumn "season_id" (strColumn "SEASON_START_ID" shotchart_202122_small |> strs)
                << dataColumn "shot_made" (numColumn "SHOT_MADE_FLAG" shotchart_202122_small |> nums)

        ps =
            params
                << param "myGrid" [ paSelect seInterval [], paBindScales ]
                << param "yearSel_second"
                    [ paValue (num 2016)
                    , paSelect sePoint [ seFields [ "season_id" ] ]
                    , paBind (ipRange [ inName "Second Player - Season Year Start", inMin 2016, inMax 2021, inStep 1 ])
                    ]
                << param "yearSel"
                    [ paValue (num 2016)
                    , paSelect sePoint [ seFields [ "season_id" ] ]
                    , paBind (ipRange [ inName "First Player or All - Season Year Start", inMin 2016, inMax 2021, inStep 1 ])
                    ]
                << param "myPlayerTwoSelect"
                    [ paBind
                        (ipSelect
                            [ inName "Select Second Player:"
                            , inOptions ([ "None" ] ++ player_options)
                            ]
                        )
                    , paSelect sePoint [ seFields [ "player_name" ] ]
                    , paValue (dataObject [ ( "player_name", str "None" ) ])
                    ]
                << param "myPlayerSelect"
                    [ paBind
                        (ipSelect
                            [ inName "Select First Player:"
                            , inOptions ([ "All" ] ++ player_options)
                            ]
                        )
                    , paSelect sePoint [ seFields [ "player_name" ] ]
                    , paValue (dataObject [ ( "player_name", str "All" ) ])
                    ]
                << param "colorSel"
                    [ paBind
                        (ipRadio
                            [ inName "Select Colouring:"
                            , inOptions
                                [ "Shot Zone", "Shot Zone Basic", "Shot Made" ]
                            ]
                        )
                    , paValue (str "Shot Zone")
                    ]

        trans =
            transform
                << VegaLite.filter (fiExpr "if(myPlayerSelect_player_name=='All', true, if(myPlayerTwoSelect_player_name == 'None', myPlayerSelect_player_name == datum.player_name, myPlayerSelect_player_name == datum.player_name || myPlayerTwoSelect_player_name == datum.player_name))")
                << VegaLite.filter (fiExpr "if(myPlayerSelect_player_name=='All', datum.season_id == yearSel.season_id, if(myPlayerTwoSelect_player_name=='None', datum.season_id == yearSel.season_id, ((datum.player_name == myPlayerSelect_player_name && datum.season_id == yearSel.season_id) || (datum.player_name == myPlayerTwoSelect_player_name && datum.season_id == yearSel_second.season_id))))")
                << calculateAs "if(colorSel == 'Shot Zone', datum.shotzone_area, if(colorSel == 'Shot Zone Basic',datum.shotzone_area_basic, datum.shot_made))" "selField"

        enc =
            encoding
                << position X
                    [ pName "x_pos"
                    , pQuant
                    , pAxis []
                    , pScale [ scDomain (doNums [ -300, 300 ]) ]
                    ]
                << position Y
                    [ pName "y_pos"
                    , pQuant
                    , pAxis
                        [ axGrid True
                        , axLabels False
                        , axTitle ""
                        , axTicks False
                        ]
                    , pScale [ scDomain (doNums [ -50, 700 ]) ]
                    ]
                << opacity
                    [ mCondition (prTest (expr "datum.shot_made==1")) [ mNum 0.9 ] [ mNum 0.3 ] ]
                << tooltips [ [ tName "player_name" ], [ tName "shot_made" ], [ tName "shotzone_area" ], [ tName "shotzone_area_basic" ] ]
                << color [ mName "selField", mLegend [] ]
                << shape [ mCondition (prTest (expr "myPlayerTwoSelect_player_name!='None' && myPlayerSelect_player_name!='All'")) [ mName "player_name", mLegend [], mScale [ scRange (raStrs [ "cross", "triangle" ]) ] ] [ mLegend [] ] ]

        shot_location_spec =
            asSpec
                [ height 800
                , width 600
                , data []
                , enc []
                , ps []
                , point [ maFilled True, maSize 70, maShape symCross ]
                , trans []
                ]

        enc_shotzone_count =
            encoding
                << position X
                    [ pName "player_name"
                    , pNominal
                    , pAxis [ axTitle "", axDomainOpacity 0.5 ]
                    ]
                << position Y
                    [ pName "shot_made"
                    , pQuant
                    , pAggregate opCount
                    , pAxis
                        [ axGrid False
                        , axTitle ""
                        , axTicks False
                        , axDomain False
                        , axLabelAlign haCenter
                        , axLabelPadding 10
                        ]
                    ]
                << color
                    [ mName "shotzone_area", mScale shotzone_colors, mLegend [] ]
                << column [ fName "shotzone_area", fSpacing 10, fHeader [ hdTitle "Count of Shots taken by Shotzone" ] ]
                << tooltips [ [ tName "shot_made", tAggregate opCount ], [ tName "player_name" ] ]

        shotzone_count_spec =
            asSpec
                [ height 70
                , width 140
                , data []
                , enc_shotzone_count []
                , ps []
                , bar []
                , trans []
                ]

        enc_shotzone_eff =
            encoding
                << position X
                    [ pName "player_name"
                    , pNominal
                    , pAxis [ axTitle "", axDomainOpacity 0.5 ]
                    ]
                << position Y
                    [ pName "shot_made"
                    , pQuant
                    , pAggregate opMean
                    , pAxis
                        [ axValues (nums [ 0, 0.5, 1 ])
                        , axGrid False
                        , axTitle ""
                        , axTicks False
                        , axLabelExpr "datum.value * 100 + '%'"
                        , axDomain False
                        , axLabelAlign haCenter
                        , axLabelPadding 10
                        ]
                    ]
                << color
                    [ mName "shotzone_area", mScale shotzone_colors, mLegend [] ]
                << column [ fName "shotzone_area", fSpacing 10, fHeader [ hdTitle "Efficieny by Shotzone" ] ]
                << tooltips [ [ tName "shot_made", tAggregate opMean ], [ tName "player_name" ] ]

        shotzone_eff_spec =
            asSpec
                [ height 70
                , width 140
                , data []
                , enc_shotzone_eff []
                , ps []
                , bar []
                , trans []
                ]

        enc_shotzone_basic_count =
            encoding
                << position X
                    [ pName "player_name"
                    , pNominal
                    , pAxis [ axTitle "", axDomainOpacity 0.5 ]
                    ]
                << position Y
                    [ pName "shot_made"
                    , pQuant
                    , pAggregate opCount
                    , pAxis
                        [ axGrid False
                        , axTitle ""
                        , axTicks False
                        , axDomain False
                        , axLabelAlign haCenter
                        , axLabelPadding 10
                        ]
                    ]
                << color
                    [ mName "shotzone_area_basic", mScale shotzone_colors, mLegend [] ]
                << column [ fName "shotzone_area_basic", fSpacing 10, fHeader [ hdTitle "Count of Shots taken by Shotzone Basic" ] ]
                << tooltips [ [ tName "shot_made", tAggregate opCount ], [ tName "player_name" ] ]

        shotzone_count_basic_spec =
            asSpec
                [ height 70
                , width 140
                , data []
                , enc_shotzone_basic_count []
                , ps []
                , bar []
                , trans []
                ]

        enc_shotzone_basic_eff =
            encoding
                << position X
                    [ pName "player_name"
                    , pNominal
                    , pAxis [ axTitle "", axDomainOpacity 0.5 ]
                    ]
                << position Y
                    [ pName "shot_made"
                    , pQuant
                    , pAggregate opMean
                    , pAxis
                        [ axValues (nums [ 0, 0.5, 1 ])
                        , axGrid False
                        , axTitle ""
                        , axTicks False
                        , axLabelExpr "datum.value * 100 + '%'"
                        , axDomain False
                        , axLabelAlign haCenter
                        , axLabelPadding 10
                        ]
                    ]
                << column [ fName "shotzone_area_basic", fSpacing 10, fHeader [ hdTitle "Efficieny by Shotzone Basic" ] ]
                << color [ mName "shotzone_area_basic", mScale shotzone_colors, mLegend [] ]
                << tooltips [ [ tName "shot_made", tAggregate opMean ], [ tName "player_name" ] ]

        shotzone_basic_eff_spec =
            asSpec
                [ height 70
                , width 140
                , data []
                , enc_shotzone_basic_eff []
                , ps []
                , bar []
                , trans []
                ]

        profile_spec =
            asSpec [ vConcat [ shotzone_count_spec, shotzone_eff_spec, shotzone_count_basic_spec, shotzone_basic_eff_spec ] ]
    in
    toVegaLite
        [ data []
        , ps []
        , cfg []
        , title
            "Indiana Pacers"
            [ tiFontSize 60
            , tiColor "black"
            , tiSubtitle "An overview of the Indiana Pacers during 2011-2022"
            , tiSubtitleFontSize 30
            , tiSubtitleColor "black"
            , tiSubtitlePadding 20
            ]
        , hConcat [ shot_location_spec, profile_spec ]
        ]
```
