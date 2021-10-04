# Simple Embed
Trigger : Command → `se`

```go
{{/*
	This is just like the simpleembed command. The difference being that is allows you to add fields to the embed.
	Suggested trigger type: Command
	Suggested trigger: embed

	Example of usage: -embed -color 1752220 -title i am the title -desc i am the description -channel 693662119765344257 -fields /name field name /value 1540 /inline true -fields /name field name 2 /value valueeee /inline true -fields /name vou ficar em baixo /value 4861 -image https://cdn.discordapp.com/attachments/682204005723799553/694677663717261402/PokecordSpawn.jpg -thumb https://cdn.discordapp.com/attachments/693662119765344257/698088081718247504/PokecordSpawn.jpg -author /name hello there /icon https://cdn.discordapp.com/attachments/693662119765344257/698087514484768818/PokecordSpawn.jpg -footer /text hello /icon https://cdn.discordapp.com/attachments/693662119765344257/698078976727318558/PokecordSpawn.jpg -timestamp

	Every flag (title, color, description, etc) should start with an -
		Example: -embed -color 1752220 -title i am the title -desc i am the description
	Every field argument (value, name, inline) should start with an /
		Example: -embed -fields /name field name /value 1540 /inline true -fields /name field name 2 /value valueeee /inline true
*/}}

{{/* ACTUAL CODE DONT TOUCH */}}
{{$multipliers := cslice 1048576 65536 4096 256 16 1}}
{{$hex2dec := sdict "A" 10 "B" 11 "C" 12 "D" 13 "E" 14 "F" 15}}

{{$capture := false}} {{$field := sdict}} {{$name := false}} {{$value := false}} {{$boolean := false}} {{$hasField := false}} {{$nameV := ""}} {{$valueV := ""}} {{$booleanV := false}} {{$color := false}} {{$colorV := 123456}} {{$fields := cslice}} {{$isEmbed := false}} {{$description := false}} {{$descriptionV := ""}} {{$channel := false}} {{$channelV := .Channel.ID}} {{$title := false}} {{$titleV := ""}} {{$image := false}} {{$imageV := sdict}} {{$thumbnail := false}} {{$thumbnailV := sdict}} {{$author := false}} {{$authorV := sdict}} {{$authorName := false}} {{$authorNameV := ""}} {{$authorIcon := false}} {{$footer := false}} {{$footerV := sdict}} {{$footerText := false}} {{$footerIcon := false}} {{$footerTextV := ""}} {{$timeStamp := false}} {{$embed := sdict}}
{{$flags := cslice "-channel" "-fields" "-color" "-desc" "-title" "-image" "-thumb" "-author" "-footer" "-timestamp"}}
{{- range $k, $v := .CmdArgs -}}
	{{- if eq . "-fields"}} {{$capture = true}} {{else if in $flags .}} {{$capture = false}} {{end -}}
	{{- if $capture -}}
		{{- $hasField = true -}}
		{{- if eq . "/name"}} {{$name = true}} {{$value = false}} {{$boolean = false -}}
		{{- else if eq . "/value"}} {{$name = false}} {{$value = true}} {{$boolean = false -}}
		{{- else if eq . "/inline"}} {{$name = false}} {{$value = false}} {{$boolean = true -}}
		{{- end -}}
		{{- if and ($name) (not (eq . "/name"))}} {{$nameV = joinStr " " $nameV .}} {{$field.Set "name" $nameV -}}
		{{- else if and ($value) (not (eq . "/value")) }} {{$valueV = joinStr " " $valueV .}} {{$field.Set "value" $valueV -}}
		{{- else if $boolean}} {{if eq . "true"}} {{$booleanV = true}} {{end}} {{$field.Set "inline" $booleanV -}}
		{{- else}} {{$field.Set "inline" $booleanV -}}
		{{- end -}}
	{{- end -}}
	{{- if and (ne $valueV "") (or (and ($hasField) (not $capture)) (and ($hasField) (eq $k (sub (len $.CmdArgs) 1))) (and (eq . "-fields") ($field.Get "name")))}} {{$hasField = false}} {{$isEmbed = true}} {{$fields = $fields.Append $field}} {{$field = sdict}} {{$nameV = ""}} {{$valueV = ""}} {{$booleanV = false}} {{end -}}
	{{- if eq . "-color"}} {{$color = true}} {{else if in $flags .}} {{$color = false}} {{end -}}
	{{- if and ($color) (not (eq . "-color"))}} {{with .}} {{$isEmbed = true}} {{$colorV = .}} {{$regex := `\A(?:#?([a-fA-F\d]{1,6}))\z`}} 
		{{with reFindAllSubmatches $regex $colorV}} 
			{{$hex := printf "%06s" (index . 0 1) | upper}} 
			{{$dec := 0}} 
			{{ range $k, $v := split $hex ""}}
				{{ $multiplier := index $multipliers $k}}
				{{ $num := or ($hex2dec.Get $v) $v }}
				{{ $dec = add $dec (mult $num $multiplier)}}
			{{end}}
			{{$colorV = $dec}}
		{{end}}
	{{- end}} {{end -}}
	{{- if eq . "-desc"}} {{$description = true}} {{else if in $flags .}} {{$description = false}} {{end -}}
	{{- if and ($description) (not (eq . "-desc"))}} {{$isEmbed = true}} {{$descriptionV = joinStr " " $descriptionV .}} {{end -}}
	{{- if eq . "-channel"}} {{$channel = true}} {{else if in $flags .}} {{$channel = false}} {{end -}}
	{{- if and ($channel) (not (eq . "-channel"))}} {{$checkChannel := reReplace `<|>|#` . ""}} {{with getChannel $checkChannel}} {{$channelV = .ID}} {{end}} {{end -}}
	{{- if eq . "-title"}} {{$title = true}} {{else if in $flags .}} {{$title = false}} {{end -}}
	{{- if and ($title) (not (eq . "-title"))}} {{$isEmbed = true}} {{$titleV = joinStr " " $titleV .}} {{end -}}
	{{- if eq . "-image"}} {{$image = true}} {{else if in $flags .}} {{$image = false}} {{end -}}
	{{- if and ($image) (not (eq . "-image"))}} {{if reFind `https?:\/\/\w+` .}} {{$isEmbed = true}} {{$imageV.Set "url" .}} {{end}} {{end -}}
	{{- if eq . "-thumb"}} {{$thumbnail = true}} {{else if in $flags .}} {{$thumbnail = false}} {{end -}}
	{{- if and ($thumbnail) (not (eq . "-thumb"))}} {{if reFind `https?:\/\/\w+` .}} {{$isEmbed = true}} {{$thumbnailV.Set "url" .}} {{end}} {{end -}}
	{{- if eq . "-author"}} {{$author = true}} {{else if in $flags .}} {{$author = false}} {{end -}}
	{{- if $author}}
		{{- if eq . "/name"}} {{$authorName = true}} {{$authorIcon = false -}}
		{{- else if eq . "/icon"}} {{$authorName = false}} {{$authorIcon = true -}}
		{{- end -}}
		{{- if and ($authorName) (not (eq . "/name"))}} {{$authorNameV = joinStr " " $authorNameV .}} {{$isEmbed = true}} {{$authorV.Set "name" $authorNameV -}}
		{{- else if and ($authorIcon) (not (eq . "/icon"))}} {{if reFind `https?:\/\/\w+` .}} {{$isEmbed = true}} {{$authorV.Set "icon_url" .}} {{end -}}
		{{- end -}}
	{{- end -}}
	{{- if eq . "-footer"}} {{$footer = true}} {{else if in $flags .}} {{$footer = false}} {{end -}}
	{{- if $footer}}
		{{- if eq . "/text"}} {{$footerText = true}} {{$footerIcon = false -}}
		{{- else if eq . "/icon"}} {{$footerText = false}} {{$footerIcon = true -}}
		{{- end -}}
		{{- if and ($footerText) (not (eq . "/text"))}} {{$footerTextV = joinStr " " $footerTextV .}} {{$isEmbed = true}} {{$footerV.Set "text" $footerTextV -}}
		{{- else if and ($footerIcon) (not (eq . "/icon"))}} {{if reFind `https?:\/\/\w+` .}} {{$isEmbed = true}} {{$footerV.Set "icon_url" .}} {{end -}}
		{{- end -}}
	{{- end -}}
	{{- if eq . "-timestamp"}} {{$timeStamp = currentTime}} {{$isEmbed = true}} {{end -}}
{{- end -}}

{{if $isEmbed}}
{{$embed.Set "fields" $fields}} {{$embed.Set "color" $colorV}} {{$embed.Set "description" $descriptionV}} {{$embed.Set "title" $titleV}} {{$embed.Set "image" $imageV}} {{$embed.Set "thumbnail" $thumbnailV}} {{$embed.Set "author" $authorV}} {{$embed.Set "footer" $footerV}} {{with $timeStamp}} {{$embed.Set "timestamp" .}} {{end}}
{{sendMessage $channelV (cembed $embed)}}
{{end}}
{{deleteTrigger 1}}
```
---

# Edit embed
Trigger : Command → `edit`

```go
{{/*
	This command is a tool for editing messages sent by YAGPDB.
	Usage: `-edit [channel] <msg> <flags...>`.

	Recommended trigger: Command trigger with trigger `edit`.
	Flags:  -content : To Change Content
		-title, -desc, -image, -thumbnail, -url, -author, -authoricon, -authorurl, -footer, -footericon, -color : To Edit Embed
		-force : Makes a new embed with provided fields and discards previous embed (default behavior is simply editing provided fields of embed while preserving other fields)
		-clrembed : To remove the embed from a message previously containing embed (so that now it has only content. Note, You cant remove embed if content is also null)
*/}}

{{$helpMsg := sdict
	"title" "`-edit [channel] <msg> <new-content>`"
	"color" 14232643
	"description" "Please provide a valid message (which was sent by YAGPDB).\n\nIf the message is an embed, you can use the syntax from the `-se` command to edit it: `-edit [channel] <msg> -title \"Hello world\" -desc \"Foobar\"`."
}}
{{$error := ""}}

{{$flags := sdict "-title" "Title" "-desc" "Description" "-url" "URL" "-image" "Image" "-thumbnail" "Thumbnail" "-author" "Author" "-authoricon" "Author" "-authorurl" "Author" "-footer" "Footer" "-footericon" "Footer" "-color" "Color" "-content" "Content" "-force" "Force" "-clrembed"  "Clear"}}
{{$subField := sdict "-image" "URL" "-thumbnail" "URL" "-author" "Name" "-authoricon" "IconURL" "-authorurl" "URL" "-footer" "Text" "-footericon" "IconURL"}}
{{$channel := .Channel}}
{{$multipliers := cslice 1048576 65536 4096 256 16 1}}
{{$hex2dec := sdict "A" 10 "B" 11 "C" 12 "D" 13 "E" 14 "F" 15}}
{{$args := cslice}}
{{$id := ""}}

{{if .CmdArgs}}
	{{$channelID := ""}}
	{{with reFindAllSubmatches `<#(\d+)>` (index .CmdArgs 0)}}{{$channelID = index . 0 1}}{{end}}
	{{$temp := getChannel (or $channelID (index .CmdArgs 0))}}
	{{if $temp}}
		{{if lt (len .CmdArgs) 3}}
			{{$error = "Insufficient number of Args"}}
		{{else}}
			{{$channel = $temp}}
			{{$id = toInt64  (index .CmdArgs 1)}}
			{{$args = slice .CmdArgs 2}}
		{{end}}
	{{else if (ge (len .CmdArgs) 2)}}
		{{$id = toInt64 (index .CmdArgs 0)}}
		{{$args = slice .CmdArgs 1}}
	{{else}}
		{{$error = "Insufficient number of Args"}}
	{{end}}
{{end}}

{{$content := ""}}{{$embed := sdict}}{{$Oembed := sdict}}{{$embedPresent := false}}{{$clear := false}}
{{if not $error}}
	{{ $msg := getMessage $channel.ID $id}}
	{{with $msg}}
		{{if eq .Author.ID 204255221017214977}}
			{{$content = .Content}}
			{{if .Embeds}}{{$embed = structToSdict (index .Embeds 0)}}
				{{range $k, $v := $embed}}{{if eq (kindOf $v true) "struct"}}{{$embed.Set $k (structToSdict $v)}}{{end}}{{end}}{{$embedPresent = true}}
			{{end}}
		{{else}}
			{{$error = "<@204255221017214977> is not Author"}}
		{{end}}
	{{else}}
		{{$error = "Unknown Message"}}
	{{end}}
{{end}}

{{if not $error}}
	{{$parseFlag := 2}}{{$currentFlag := ""}}{{$currentField := ""}}{{$skip := 0}}
	{{range $args}}
		{{- if and (not $error) (gt $parseFlag 1)}}
			{{- if ($f := $flags.Get (lower .))}}
				{{- if eq $f "Force"}}
					{{- $embed = sdict}}{{range $k,$v :=$Oembed}}{{$embed.Set $k $v}}{{end}}{{$Oembed = sdict}}{{$parseFlag = 1}}{{$currentFlag = ""}}{{$skip = 1}}
				{{- else if eq $f "Clear"}}
					{{- $embed = $.nil}}{{$parseFlag = 1}}{{$clear = true}}{{$currentFlag = ""}}{{$skip = 1}}
				{{- else if and $clear (ne $f "Content")}}
					{{- $error = print "Parsing Error: Invalid flag: " . ". Attempting to Both Clear and Edit Embed"}}
				{{- else}}
					{{- $currentFlag = $f}}{{$parseFlag = 0}}{{$currentField = $subField.Get (lower .)}}
				{{- end}}
			{{- end}}
		{{- end}}

		{{- if and (not $error) $parseFlag (not $skip)}}
			{{- if $currentFlag}}
				{{- if in (cslice "Description" "Title" "URL") $currentFlag}}
					{{- if eq $parseFlag 1}}{{$embed.Set $currentFlag ""}}{{$Oembed.Set $currentFlag ""}}{{end}}
					{{- $embed.Set $currentFlag (joinStr " " ($embed.Get $currentFlag) .)}}{{$Oembed.Set $currentFlag (joinStr " " ($Oembed.Get $currentFlag) .)}}{{$embedPresent = true}}
				{{- else if eq $currentFlag "Color"}}
					{{- if eq $parseFlag 1}}
						{{- $regex := `\A(?:#?([a-fA-F\d]{1,6}))\z`}}
						{{- with reFindAllSubmatches $regex .}}
							{{- $hex := printf "%06s" (index . 0 1) | upper}}
							{{- $dec := 0}}
							{{- range $k, $v := split $hex ""}}
								{{- $multiplier := index $multipliers $k}}
								{{- $num := or ($hex2dec.Get $v) $v }}
								{{- $dec = add $dec (mult $num $multiplier)}}
							{{- end}}
							{{- $embed.Set $currentFlag $dec}}{{$Oembed.Set $currentFlag $dec}}
						{{- else}}
							{{- $error = "Parsing Error: color was not in correct format (use hex)" }}
						{{- end}}
					{{- else}}
						{{- $error = "Parse Error: too many arguments to Color"}}
					{{- end}}
					{{- $embedPresent = true}}
				{{- else if eq $currentFlag "Content"}}{{if eq $parseFlag 1}}{{$content = ""}}{{end}}{{$content = joinStr " " $content .}}
				{{- else}}
					{{- if eq $parseFlag 1}}
						{{- $tmp :=or ($embed.Get $currentFlag) sdict}}{{$tmpO :=or ($Oembed.Get $currentFlag) sdict}}
						{{- $tmp.Set $currentField ""}}{{$tmpO.Set $currentField ""}}
						{{- $embed.Set $currentFlag $tmp}}{{$Oembed.Set $currentFlag $tmpO}}{{$embedPresent = true}}
					{{- end}}
					{{- $cFlag := $embed.Get $currentFlag}}{{$cFlagO := $Oembed.Get $currentFlag}}
					{{- $cFlag.Set $currentField (joinStr " " ($cFlag.Get $currentField) .)}}{{$cFlagO.Set $currentField (joinStr " " ($cFlagO.Get $currentField) .)}}
				{{- end}}
			{{- else}}
				{{- $error = (print "Parsing Error:  Invalid flag: " . )}}
			{{- end}}
		{{- end}}
		{{- $parseFlag = add $parseFlag 1}}{{$skip = 0}}
	{{- end}}

	{{if $embed}}
		{{if $embed.Author}}{{$embed.Author.Set "Icon_URL" $embed.Author.IconURL}}{{end}}
		{{if $embed.Footer}}{{$embed.Footer.Set "Icon_URL" $embed.Footer.IconURL}}{{end}}
		{{$embed.Set "timestamp" currentTime}}
	{{end}}
	{{if (not $embedPresent)}}{{$embed = $.nil}}{{end}}

	{{if not $error}}
		{{if or $content (ne (print $embed) "<nil>")}}
			{{editMessage $channel.ID $id (complexMessageEdit "content" $content "embed" $embed)}}
			Done :+1:
		{{else}}
			{{$error = "Content and embed cannot be null at the same time."}}
		{{end}}
	{{end}}
{{end}}

{{if $error}}
	{{$helpMsg.Set "description" (print "**Error** - `" $error  "`\n" ($helpMsg.Get "description"))}}
	{{sendMessage nil (cembed $helpMsg)}}
{{end}}
{{deleteResponse 5}}
{{deleteTrigger 1}}
```

---

# Get Message Embed
Trigger : Command → `text <id>`

```go
{{$chan := reFind `\-chan` .Message.Content}}
{{$id := 0}}
{{$channel := "nil"}}
{{$champ := reFind `\-(footer|title|field(s?)|color|author)` .Message.Content }}
{{if $chan}}
	{{$channel = (toInt (index .CmdArgs 1))}}
	{{$id = (toInt (index .CmdArgs 2)) }}
{{else}}
	{{$channel = .Channel.ID}}
	{{$id = (toInt (index .CmdArgs 0))}}
{{end}}
{{$msg:= getMessage $channel $id}}
{{$rep := ""}}
{{if $msg.Embeds}}
  {{with (index $msg.Embeds 0)}}
		{{if eq $champ "-field" "-fields"}}
			{{ range $i, $j := .Fields }} 
				{{ $rep = print $rep $j.Name " : \n ```\n" $j.Value "\n ```\n\n" }}
        {{$i = add $i 1}}
			{{ end }}
		{{else if eq $champ "-footer"}}
			{{$rep = print $rep .Footer.Text "\n"}}
		{{else if eq $champ "-title"}}
			{{$rep = print $rep .Title "\n"}}
		{{else if eq $champ "-color"}}
			{{$rep = print $rep .Color "\n"}}
		{{else if eq $champ "-author"}}
			{{$rep = print $rep .Author.Name "\n"}}
    {{else}}
    	{{$rep = print  $rep .Description "\n"}}
		{{end}}
  {{end}}
{{end}}
{{$content := ""}}
{{if $msg.Content}}
	{{$content = print $msg.Content}}
{{end}}
{{deleteTrigger 1}}
{{if (reFind `-dl` .Message.Content)}}
  {{$content = print "Voici le document demandé ! \n " $content}}
	{{$idm := sendMessageRetID nil (complexMessage "content" $content "embed" nil "file" $rep)}}
	{{deleteMessage nil $idm 180}}
{{else}}
  {{if le (len $rep) 2000}}
	  {{$idm := sendMessageRetID nil (print "```" $rep "```")}}
		{{deleteMessage nil $idm 180}}
		{{if $msg.Content}}
			{{$idc := sendMessageRetID nil (print "```" $content "```")}}
			{{deleteMessage nil $idc 180}}
		{{end}}
  {{else}}
    {{$content = print "Votre réponse est trop longue, donc voici le fichier le contenant. \n " $content}}
  	{{$idm := sendMessageRetID nil (complexMessage "content" $content "embed" nil "file" $rep)}}
    {{deleteMessage nil $idm 180}}
  {{end}}
{{end}}

```

# Cite message
Trigger : Regex → `https://(?:\w+\.)?discord(?:app)?\.com/channels\/(\d+)\/(\d+)\/(\d+)`

```go
{{$col := 16777215}}
{{$p := 0}}
{{$r := .Member.Roles}}
{{range .Guild.Roles}}
	{{if and (in $r .ID) (.Color) (lt $p .Position)}}
	{{$p = .Position}}
	{{$col = .Color}}
	{{end}}
{{end}}

{{if reFindAllSubmatches `https://(?:\w+\.)?discord(?:app)?\.com/channels\/(\d+)\/(\d+)\/(\d+)` .Message.Content}}
    {{$m := reFindAllSubmatches `https://(?:\w+\.)?discord(?:app)?\.com/channels\/(\d+)\/(\d+)\/(\d+)` .Message.Content}}
    {{if not (eq (toInt64 (index (index $m 0) 1)) .Guild.ID)}}
        {{sendMessage nil (cembed
            "author" (sdict
                "name" "Utilisateur inconnu"
                "icon_url" "https://cdn.discordapp.com/emojis/565142262401728512.png")
            "description" (print "\n\n**[➥ Lien vers l'original](" (index (index $m 0) 0) "/) to <#" (index (index $m 0) 2) ">**\n" "❗ Ce message est issue d'un autre serveur, son contenu ne peut pas être affiché ❗")
            "color" 0x36393F
            "footer" (sdict
                "text" (print "Cité par : " .Message.Author.String))
            "timestamp" ((newDate 1970 1 1 0 0 0).Add (toDuration (mult (fdiv (toInt64 (round (add (fdiv (toInt64 (index (index $m 0) 3)) 4194304) 1420070400000))) 1000) .TimeSecond))))}}
    {{else}}
        {{$msg := getMessage (index (index $m 0) 2) (index (index $m 0) 3)}}
        {{$mc := reReplace `(?:https?://)?(?:www\.)?(discord(?:app)?\.gg(?:/|\\+/+)|discord(?:app)?\.com(?:/|\\+/+)(?:invite/))[A-z+0-9]{2,}|(?:https?://)?(?:www\.)?discord(?:app)?\.(?:io|me|li)(?:/|\\+/+)[A-z+0-9]{2,}` $msg.Content "[Invitation retirée](https://discord.gg/GRns3fg)"}}
        {{$e := sdict
            "author" (sdict
                "name" (print $msg.Author.String)
                "icon_url" ($msg.Author.AvatarURL "1024"))
            "color" $col
            "footer" (sdict
                "text" (print "Cité par : " .Message.Author.String ))
            "timestamp" $msg.Timestamp}}
        {{$fo := $e.Get "footer"}}
        {{$ti := $msg.Timestamp}}
        {{if $msg.Attachments}}
            {{range $c,$ma := $msg.Attachments}}
                {{$e.Del "footer"}}
                {{$e.Del "timestamp"}}
                {{if eq $c 0}}
                    {{$e.Set "description" (print $mc "\n\n [➥ Original](" (index (index $m 0) 0) "/)")}}
                {{end}}
                {{if eq (len $msg.Attachments) (add $c 1)}}
                    {{$e.Set "footer" $fo}}
                    {{$e.Set "timestamp" $ti}}
                {{end}}
                {{if (reFind "(?i)(.jpg|.jpeg|.png|.gif|.tif|.tiff)$" .Filename)}}
                    {{$e.Set "image" (sdict "url" .URL)}}
                    {{sendMessage nil (cembed $e)}}
                {{else}}
                    {{$e.Set "fields" (cslice
                        (sdict "name" "Nom du fichier" "value" (print "`" .Filename "`") "inline" true)
                        (sdict "name" "URL" "value" (print "[Lien](" .URL ")") "inline" true))}}
                    {{sendMessage nil (cembed $e)}}
                {{end}}
                {{$e.Del "fields"}}
                {{$e.Del "image"}}
                {{$e.Del "description"}}
                {{$e.Del "author"}}
            {{end}}
        {{else}}
            {{if $msg.Content}}
							{{$e.Set "description" (print $mc "\n\n [➥ Original](" (index (index $m 0) 0) "/)")}}
              {{sendMessage nil (cembed $e)}}
            {{end}}
        {{end}}
        {{if $msg.Embeds}}
            {{sendMessage nil (index $msg.Embeds 0)}}
        {{end}}
    {{end}}
    {{if eq (len (index (index $m 0) 0)) (len .Message.Content)}}
        {{deleteTrigger 0}}
    {{end}}
{{end}}

```
---
# Create rules
Trigger : Command → `rules chan_id id_message -(add|rm|edit)`

```go
{{$chan := reFind `\-chan` .Message.Content}}
{{$id := 0}}
{{$channel := "nil"}}
{{$name := ""}}
{{$value := ""}}
{{$cmd := ""}}
{{if and $chan (ge (len .CmdArgs) 5)}}
	{{$channel = (toInt (index .CmdArgs 1))}}
	{{$id = (toInt (index .CmdArgs 2)) }}
  {{$cmd = index .CmdArgs 3}}
{{else if and (not $chan) (ge (len .CmdArgs) 3)}}
	{{$channel = .Channel.ID}}
	{{$id = (toInt (index .CmdArgs 0))}}
  {{$cmd = index .CmdArgs 1}}
{{end}}
{{$msg:= getMessage $channel $id}}
{{$field := sdict}}
{{$fields := cslice}}
{{if $msg.Embeds}}
  {{$embed := structToSdict (index $msg.Embeds 0) }}
  {{range $k, $v := $embed }}
    {{- if eq (kindOf $v true) "struct" }}
      {{- $embed.Set $k (structToSdict $v) }}
    {{- end -}}
  {{ end }}
  {{$cont := ""}}
  {{if $msg.Content}}
    {{$cont = print $msg.Content}}
  {{end}}
  {{if eq $msg.Author.ID 204255221017214977}}
    {{if eq $cmd "add" "-add"}}
      {{if and $chan (ge (len .CmdArgs) 5)}}
        {{$name = index .CmdArgs 4}}
        {{$value = index .CmdArgs 5}}
      {{else if and (not $chan) (ge (len .CmdArgs) 3)}}
        {{$name = index .CmdArgs 2}}
        {{$value = index .CmdArgs 3}}
      {{end}}
      {{if (ne $name "" )}}
        {{with (index $msg.Embeds 0)}}
          {{if .Fields }}
            {{range $i, $j := .Fields}}
              {{$field.Set "name" $j.Name}}
              {{$field.Set "value" $j.Value}}
              {{$fields = $fields.Append $field}}
              {{$field = sdict}}
            {{end}}
          {{end}}
        {{end}}
        {{$f := sdict}}
        {{if eq $value ""}}
          {{$value = "_ _"}}
        {{end}}
        {{$f.Set "value" $value}}
        {{$f.Set "name" $name}}
        {{$fields = $fields.Append $f}}
        {{$embed.Set "fields" $fields}}
        {{$embed = cembed $embed}}
        {{editMessage $channel $id (complexMessageEdit "content" $cont "embed" $embed)}}
        Done !
      {{end}}
    {{else if eq $cmd "-remove" "-rm"}}
      {{if and $chan (ge (len .CmdArgs) 4)}}
        {{$name = index .CmdArgs 4}}
      {{else if and (not $chan) (ge (len .CmdArgs) 2)}}
        {{$name = index .CmdArgs 2}}
      {{end}}
      {{with (index $msg.Embeds 0)}}
        {{if .Fields}}
          {{range $i, $j := .Fields}}
            {{if ne (title $j.Name) (title $name)}}
              {{$field.Set "name" $j.Name}}
              {{$field.Set "value" $j.Value}}
              {{$fields = $fields.Append $field}}
              {{$field = sdict}}
            {{else if eq (title $j.Name) (title $name)}}
		{{$idS := sendMessageRetID nil (print "`" ($j.Name) "` a été supprimé")}}
		{{deleteMessage nil $idS 15}}
            {{end}}
          {{end}}
        {{end}}
      {{end}}
        {{$embed.Set "fields" $fields}}
        {{$embed = cembed $embed}}
        {{editMessage $channel $id (complexMessageEdit "content" $cont "embed" $embed)}}
        Done !
    {{else if eq $cmd "-edit" "-ed"}}
      {{if and $chan (ge (len .CmdArgs) 5)}}
        {{$name = index .CmdArgs 4}}
        {{$value = index .CmdArgs 5}}
      {{else if and (not $chan) (ge (len .CmdArgs) 3)}}
        {{$name = index .CmdArgs 2}}
        {{$value = index .CmdArgs 3}}
      {{end}}
      {{with (index $msg.Embeds 0)}}
        {{if .Fields}}
          {{range $i, $j := .Fields}}
            {{if ne (title $j.Name) (title $name)}}
              {{$field.Set "name" $j.Name}}
              {{$field.Set "value" $j.Value}}
              {{$fields = $fields.Append $field}}
            {{else if eq (title $j.Name) (title $name)}}
		{{$idS := sendMessageRetID nil (print "`" $j.Name "` a été édité")}}
		{{deleteMessage nil $idS 15}}
              {{$field.Set "name" $j.Name}}
              {{$field.Set "value" $value}}
              {{$fields = $fields.Append $field}}
            {{end}}
            {{$field = sdict}}
          {{end}}
        {{end}}
      {{end}}
      {{$embed.Set "fields" $fields}}
      {{$embed = cembed $embed}}
      {{editMessage $channel $id (complexMessageEdit "content" $cont "embed" $embed)}}
      Done !
    {{end}}
  {{end}}
{{end}}
{{deleteTrigger 1}}
{{deleteResponse 15}}
```
---
# Purge
Trigger : Command → `purge`

```go
	{{execAdmin "clean" "100"}}
	{{execAdmin "clean" "100"}}
	{{execAdmin "clean" "100"}}
	{{execAdmin "clean" "100"}}
	{{execAdmin "clean" "100"}}
Nettoyé !
{{deleteTrigger 1}}
{{deleteResponse 30}}
```

# Reaction
(Add reaction to a message)
Trigger : Command → `reaction <emoji>` 

```go
{{if .CmdArgs}}
  {{$id := 0}}
  {{$emo := ""}}
  {{$id = (index .CmdArgs 0)}}
  {{$emo = (slice .CmdArgs 1)}}
  {{range $emo}}
    {{addMessageReactions nil $id .}}
  {{end}}
{{end}}
{{deleteTrigger 1}}
```
---
# Meteo in french (french traduction of the api for YAGPDB)
Command → `mt`
Interval → Every 4 hours 

```go
{{$city := "Villefranche-sur-mer"}}
{{$rename := "Nibelhiem"}}
{{$x:= exec "weather" $city}} 
{{$x = reReplace $city $x $rename}}
{{$x = reReplace "Weather report" $x "Bulletin Météo "}}
{{$x = reReplace `\(\d+ °F\)` $x ""}}
{{$x = reReplace `\.\.` $x " ►►► "}}
{{$meteo := sdict "Rain" "Pluie" "Light Rain" "Pluie légère" "Rain Shower" "Averses" "Clear" "Clair" "Sunny" "Ensoleillé" "Partly cloudy" "Partiellement couvert" "Cloudy" "Nuageux" "Overcast" "Couvert" "Mist" "Brumeux" "Patchy rain possible" "Possible pluie fine" "Patchy snow possible" "Possibles chute de neiges irrégulières" "Patchy sleet possible" "Possibles chutes irrégulières de giboulées" "Patchy freezing drizzle possible" "Possible bruine verglaçante" "Thundery outbreaks possible" "Possibles orages" "Blowing snow" "Poudrerie" "Blizzard" "Blizzard" "Fog" "Brouillard" "Freezing fog" "Brouillard givrant" "Patchy light drizzle" "Légères bruines irrégulières" "Light drizzle" "Légère bruine" "Freezing drizzle" "Bruine givrante" "Heavy freezing drizzle" "Forte bruine givrante" "Patchy light rain" "Légères pluies irrégulières" "Light rain" "Légère pluie" "Moderate rain at times" "Pluie modérée intermittente" "Moderate rain" "Pluie modérée" "Heavy rain at times" "Forte pluie intermittente" "Heavy rain" "Forte pluie" "Light freezing rain" "Légère pluie verglaçante" "Moderate or heavy freezing rain" "Pluie verglaçante modérée à forte" "Light sleet" "Légère giboulées" "Moderate or heavy sleet" "Chutes de giboulées modérées à fortes" "Patchy light snow" "Légères neiges irrégulières" "Light snow" "Légère neige" "Patchy moderate snow" "Neiges irrégulières modérées" "Moderate snow" "Neiges modérées" "Patchy heavy snow" "Irrégulière neige abondante" "Heavy snow" "Neige abondante" "Ice pellets" "Grésil" "Light rain shower" "Légères averses" "Moderate or heavy rain shower" "Averses modérées à fortes" "Torrential rain shower" "Averses torrentielles" "Light sleet showers" "Légères averses de giboulées" "Moderate or heavy sleet showers" "Averses de giboulées modérées à fortes" "Light snow showers" "Légères averses de neige" "Moderate or heavy snow showers" "Averses de neige modérées à fortes" "Patchy light rain with thunder" "Irrégulière légères pluies orageuses" "Moderate or heavy rain with thunder" "Pluies orageuses modérées à fortes" "Patchy light snow with thunder" "Irrégulière légères neiges orageuses" "Moderate or heavy snow with thunder" "Neige orageuses modérées à fortes" "Snow" "Neige" "Mist" "Brouillard"}}
{{$y := reFind `(Rain|Light Rain|Rain Shower|Clear|Sunny|Partly cloudy|Cloudy|Overcast|Mist|Patchy rain possible|Patchy snow possible|Patchy sleet possible|Patchy freezing drizzle possible|Thundery outbreaks possible|Blowing snow|Blizzard|Fog|Freezing fog|Patchy light drizzle|Light drizzle|Freezing drizzle|Heavy freezing drizzle|Patchy light rain|Light rain|Moderate rain at times|Moderate rain|Heavy rain at times|Heavy rain|Light freezing rain|Moderate or heavy freezing rain|Light sleet|Moderate or heavy sleet|Patchy light snow|Light snow|Patchy moderate snow|Moderate snow|Patchy heavy snow|Heavy snow|Ice pellets|Light rain shower|Moderate or heavy rain shower|Torrential rain shower|Light sleet showers|Moderate or heavy sleet showers|Light snow showers|Moderate or heavy snow showers|Patchy light rain with thunder|Moderate or heavy rain with thunder|Patchy light snow with thunder|Moderate or heavy snow with thunder|Snow|Mist)` $x}}
{{if ($meteo.Get $y)}}
  {{$x = reReplace $y $x ($meteo.Get $y)}}
{{end}}
{{$x}}
{{deleteTrigger 1}}
```
---
# End
Command → `end`

```go
``` ```
{{deleteTrigger 1}}
```

---
# Delete HRP

## Delete HRP 
Regex → `^\((.*)?\)?`

```go
{{$cat := .Channel.ParentID}}
{{$roleCat := (dbGet 0 "Role_Category").Value}}
{{if not (dbGet 0 "Role_Category") }}
	{{$roleCat = cslice}}
{{else}}
{{$roleCat = (dbGet 0 "Role_Category").Value}}
 {{end}}
{{if (eq (in $roleCat $cat) true) or (eq (in $roleCat (toString $cat)) true) }}
	{{ $matches := reFindAllSubmatches `^\((.*)?\)?` .Message.Content }}
	{{if eq (len (index (index $matches 0) 0)) (len .Message.Content) }}
		{{deleteTrigger 180}}
	{{end}} 
{{end}}
```
## Config HRP channel - categories

Regex → `\$rmc \+`
Usage : `$rmc +channelID`
```go
{{$cat := .Channel.ParentID}}
{{$roleCat := cslice}}
{{if not (dbGet 0 "Role_Category") }}
    {{$roleCat = cslice}}
{{else}}
    {{$old_cat := (dbGet 0 "Role_Category").Value}}
    {{range $old_cat}}
        {{$roleCat = $roleCat.Append .}}
    {{end}}
 {{end}}

{{if .CmdArgs}}
    {{$cat = .CmdArgs}}
    {{if ge (len $cat ) 2}}
        {{range $cat}}
            {{if eq (in $roleCat .) false}}
                {{$roleCat = $roleCat.Append (toInt .)}}
            {{end}}
        {{end}}
    {{else}}
        {{$cat = toInt (index .CmdArgs 1)}}
        {{if eq (in $roleCat $cat) false}}
            {{$roleCat = $roleCat.Append $cat}}
        {{end}}
    {{end}}
{{else}}
    {{if eq (in $roleCat $cat) false}}
        {{$roleCat = $roleCat.Append $cat}}
    {{end}}
{{end}}

{{dbSet 0 "Role_Category" $roleCat}}
{{$m := sendMessageRetID nil (print $cat " est maintenant dans la base de donnée")}}
{{deleteTrigger 1}}
{{deleteMessage nil $m 30}}
```
---
# Remove channel
Regex → `\$rmc \-`
Usage : `$rmc -channelID`
```go
{{$cat := .Channel.ParentID}}
{{$roleCat := cslice}}
{{if not (dbGet 0 "Role_Category") }}
    {{$roleCat = cslice}}
{{else}}
    {{$old_cat := (dbGet 0 "Role_Category").Value}}
    {{range $old_cat}}
        {{$roleCat = $roleCat.Append .}}
    {{end}}
 {{end}}

 {{$new_cat := cslice}}

 {{if .CmdArgs}}
    {{$cat = .CmdArgs}}
    {{if ge (len $cat ) 2}}
        {{range $cat}}
            {{if eq (in $roleCat .) true}}
                {{$new_cat = $new_cat.Append .}}
            {{end}}
        {{end}}
    {{else}}
        {{$cat = toInt (index .CmdArgs 1)}}
        {{if eq (in $roleCat $cat) true}}
            {{$new_cat = $new_cat.Append .}}
        {{end}}
    {{end}}
{{else}}
    {{if eq (in $roleCat $cat) true}}
        {{$new_cat = $new_cat.Append .}}
    {{end}}
{{end}}

{{dbSet 0 "Role_Category" $new_cat}}
{{$m := sendMessageRetID nil (print $cat " a été retiré de la base de donnée")}}
{{deleteTrigger 1}}
{{deleteMessage nil $m 30}}
```
## Auto delete Emoji in RP channel
```
{{$cat := .Channel.ParentID}}
{{$roleCat := (dbGet 0 "Role_Category").Value}}
{{if not (dbGet 0 "Role_Category") }}
	{{$roleCat = cslice}}
{{else}}
{{$roleCat = (dbGet 0 "Role_Category").Value}}
 {{end}}
{{if (eq (in $roleCat $cat) true) or (eq (in $roleCat (toString $cat)) true) }}
	{{ $matches := reFindAllSubmatches `(((<a?:[\w~]{2,32}:\d{17,19}>)|[\x{1f1e6}-\x{1f1ff}]{2}|\p{So}\x{fe0f}?[\x{1f3fb}-\x{1f3ff}]?(\x{200D}\p{So}\x{fe0f}?[\x{1f3fb}-\x{1f3ff}]?)*|[#\d*]\x{FE0F}?\x{20E3})((\s+)?)(\<\#\d{17,}\>)?((\s+)?)(>\S+)?)` .Message.Content }}	
	{{if eq (len (index (index $matches 0) 0)) (len .Message.Content) }}
		{{deleteTrigger 1}}
	{{end}}
{{end}}
```
---
# Ticket - Message with reaction

# Create channel
Trigger → On reaction : added reaction only
```go
{{$ticketmsg := toInt (dbGet .Channel.ID "ticketmsg").Value}}
{{$ticketEmbed := 823671899594031204}}
{{if and (eq .Message.ID $ticketEmbed) (eq .Reaction.Emoji.Name "📝")}}
  {{$user := .Member.Nick}}
  {{if eq (len $user) 0}}
    {{$user = .User.Username}}
  {{end}}
  {{exec "ticket open" (print "fiche " $user)}}
  {{deleteMessageReaction nil $ticketEmbed .User.ID "📝"}}
{{else if and (eq .Reaction.Emoji.Name "🗑️") (eq (toInt .Reaction.MessageID) $ticketmsg) }}
  {{exec "tickets close"}}
{{else if and (eq .Reaction.Emoji.Name "📑") (eq (toInt .Reaction.MessageID) $ticketmsg) }}
  {{sendDM (cembed "title" (print "Retranscription pour " .Channel.Name "!") "description" (print "[**Click**](" (execAdmin "logs") ")" ))}}
  {{.User.Mention}} Va voir dans tes DM !
  {{deleteMessageReaction nil .Reaction.MessageID .Reaction.UserID `📑`}}
{{end}}
{{deleteResponse 0}}
```
# Message on created channel
```go
{{$id := sendMessageRetID nil (print "✅ " .User.Mention " Ton channel a été créée ! Tu peux noter tout ce que tu souhaites, poser tes questions... Et quand tu auras fini, envoie nous ta fiche ! \n \n Quand tu considères que tout est validé, tu peux fermer ton channel en réagissant à ce message avec 🗑️. \n\n <:next:780790919297368096> Au besoin, tu peux rajouter quelqu'un sur le channel avec `$ticket adduser @mention` \n <:next:780790919297368096> Tu peux retrouver une retranscription (les logs) du channel à l'aide de l'émoji 📑") }}
{{sendMessageNoEscape 786344149472903168 (print " <@&786283841701937192> !\n" .User.Mention " a commencé sa fiche dans <#" .Channel.ID "> !\n  N'oubliez pas d'y jeter un coup d'oeil !")}}
{{addMessageReactions nil $id "🗑️" "📑"}}
{{$msget := getMessage nil $id}}
{{dbSet .Channel.ID "ticketmsg" (toString $msget.ID)}}
{{editChannelName nil (reReplace `\d+` (getChannel nil).Name "")}}
```

---

# Âge verification
trigger → command `age`
Usage : `$age <nombre>`

```
{{$args := parseArgs 1 "La commande est `$age nb` où  tu remplaces`nb` par ton âge en **chiffre** !"
	(carg "int" "age")}}
{{$age:= $args.Get 0}}
{{if lt $age 15}}
	{{sendDM "Vous ne pouvez pas rejoindre le RP car vous n'avez pas l'âge requis !"}}
	{{sendMessage 808837717831712798 (print .User " n'a pas l'âge requis pour rejoindre le RP !")}}
{{else}}
	{{$role := cslice 832582454539059201 808837717491712035 808837717491712031}} 
		{{range $role}}
			{{- addRoleID .}}
    		{{- end}}
		{{removeRoleID 833763903649480725 }}
	{{sendDM "Bienvenue sur le serveur !"}}
{{end}}
{{deleteTrigger 1}}
{{deleteResponse 1}}
```

---
# Compendium
## New compendium

Regex → `^(?i)(\*?\*?)(nom(:?))`
```go
{{$regTit := `(?im)^(Nom(:?)(.*))$`}}
{{if (not (reFind $regTit .Message.Content))}}
  {{$id := sendMessageRetID nil "Votre message n'est pas au bon format ! Vous savez trois minutes pour le récupérer avant sa suppression. Vous devez le REPOSTER. Pas l'éditer. Je ne lis pas les éditions."}}
  {{$idT := sendMessageRetID nil "Le format est le suivant : \n```md\n Nom : \n Type : \n Description : \n Effet : \n (lien vers une image, facultatif)``` "}}
  {{deleteMessage nil $idT 180}}
  {{deleteMessage nil $id 180}}
  {{deleteTrigger 180}}
{{else}}
  {{$col := 0xC7DDE4}}
  {{$name := reFind $regTit .Message.Content}}
  {{$name = reReplace `^(?im)nom\W?:?` $name ""}}
  {{$desc := reReplace $regTit .Message.Content ""}}
{{$desc = reReplace `(?im)^(type)(\W*:)` $desc "**__Type__ :**"}}
{{$desc = reReplace `(?im)^(description|desc)(\W*:)` $desc "\n"}}
{{$desc = reReplace `(?im)^(effet)(\W*:)` $desc "\n**__Effets connus__ :**"}}
 {{$img := reFind `((http(s?):\/\/)(.*)\.(png|jpg|gif|jpeg|gifv))` $desc}}
  {{$col = 0x8F7DDA}}
  {{$icon := "https://i.imgur.com/z7oWZOC.png"}}
  {{$msg := cembed
  "title" (upper $name)
  "description" $desc
  "footer" (sdict "text" (print "Proposé par " .User.Username) "icon_url" ((.User.AvatarURL "256")))
  "color" $col}}
  {{$embed := structToSdict $msg}}
  {{ range $k, $v := $embed}}
     {{- if eq (kindOf $v true) "struct" }}
       {{- $embed.Set $k (structToSdict $v) }}
     {{- end -}}
  {{ end }}
  {{if $img}}
    {{$desc = reReplace $img $desc ""}}
    {{$embed.Set "image" (sdict "url" $img)}}
    {{$embed.Set "description" $desc}}
  {{end}}
  {{$idE := sendMessageRetID nil (cembed $embed)}}
  {{addMessageReactions nil $idE "✅" "*️⃣" "❌"}}
  {{deleteTrigger 30}}
{{end}}
```
## Reaction - Validation
Reaction → Added reaction only

```go
{{$occ := 814206096528900107}}
{{$chan := 814208095537725460}}
{{$staff := 808837717520547881}}
{{if eq .Channel.ID $chan}}
  {{if .ReactionAdded}}
    {{$embed := structToSdict (index .Message.Embeds 0) }}
    {{ range $k, $v := $embed }}
      {{- if eq (kindOf $v true) "struct" }}
        {{- $embed.Set $k (structToSdict $v) }}
      {{- end -}}
    {{ end }}
    {{$embed.Set "title" (upper $embed.Title)}}
    {{$tex := reFind .User.Username $embed.Footer.Text}}
    {{if and (eq .Reaction.Emoji.Name "🗑️") $tex}}
      {{deleteMessage nil .Message.ID 0}}
      {{print "Message supprimé"}}
      {{deleteResponse 10}}
    {{else if and (eq .Reaction.Emoji.Name "📥") (hasRoleID $staff)}}
      {{$embed.Set "Footer" (sdict "text" (print "Validé par " (title .User.Username)) "Icon_URL" (.User.AvatarURL "256"))}}
      {{$embed.Set "color" 9403866}}
      {{sendMessage $occ (cembed $embed)}}
      {{addMessageReactions nil .Message.ID "🎉"}}
      {{deleteMessage nil .Message.ID 30}}
    {{else if and (eq .Reaction.Emoji.Name "⛔") (hasRoleID $staff)}}
      {{$embed.Set "color" 14492194}}
      {{$embed.Footer.Set "text" $embed.Author.Name}}
      {{$embed.Author.Set "name" (print "REFUSÉ PAR " (upper .User.Username))}}
      {{editMessage nil .Message.ID (complexMessageEdit "content" "" "embed" (cembed $embed))}}
      {{deleteAllMessageReactions nil .Message.ID}}
    {{end}}
  {{end}}
{{end}}
```

## Comment compendium
Regex → `\$(comment|comm|com|decom|decomm|decomment|rmcom|comrm)`
Usage `$command <id> Reason`

```go
{{ $fieldName := printf "Commentaire de %s" .User.String }}
{{$fields := cslice}}
{{$field := sdict}}
{{$staff := 808837717520547881}}
{{if hasRoleID $staff}}
  {{if eq .Cmd "$comment" "$comm" "$com" }}
    {{if eq (len .CmdArgs) 2}}
      {{ $reason :=  joinStr "" (slice .CmdArgs 1)}}
      {{$id := index .CmdArgs 0}}
      {{$msg := getMessage nil $id}}
      {{if $msg.Embeds }}
        {{$embed := structToSdict (index $msg.Embeds 0) }}
        {{ range $k, $v := $embed }}
          {{- if eq (kindOf $v true) "struct" }}
            {{- $embed.Set $k (structToSdict $v) }}
          {{- end -}}
        {{ end }}
        {{with (index $msg.Embeds 0)}}
          {{if .Fields }}
            {{range $i, $j := .Fields}}
              {{$field.Set "name" $j.Name}}
              {{$field.Set "value" $j.Value}}
              {{$fields = $fields.Append $field}}
              {{$field = sdict}}
            {{end}}
          {{end}}
        {{end}}
        {{$f := sdict}}
        {{$f.Set "name" $fieldName}}
        {{$f.Set "value" $reason}}
        {{$fields = $fields.Append $f}}
        {{$embed.Set "fields" $fields}}
        {{$embed = cembed $embed}}
        {{editMessage nil $id (complexMessageEdit "content" "" "embed" $embed)}}
      {{end}}
    {{end}}
  {{else if eq .Cmd "$decom" "$decomment" "$rmcom" "$comrm" "$decomm" }}
    {{if eq (len .CmdArgs) 1}}
      {{$id := index .CmdArgs 0}}
      {{$msg := getMessage nil $id}}
      {{$user := .User.String}}
      {{if $msg.Embeds }}
        {{$embed := structToSdict (index $msg.Embeds 0) }}
        {{ range $k, $v := $embed }}
          {{- if eq (kindOf $v true) "struct" }}
            {{- $embed.Set $k (structToSdict $v) }}
          {{- end -}}
        {{ end }}
        {{with (index $msg.Embeds 0)}}
          {{if .Fields }}
            {{range $i, $j := .Fields}}
              {{if not (reFind $user $j.Name)}}
                {{$field.Set "name" $j.Name}}
                {{$field.Set "value" $j.Value}}
                {{$fields = $fields.Append $field}}
                {{$field = sdict}}
              {{else}}
              {{end}}
            {{end}}
          {{end}}
        {{end}}
        {{$embed.Set "fields" $fields}}
        {{$embed = cembed $embed}}
        {{editMessage nil $id (complexMessageEdit "content" "" "embed" $embed)}}
      {{end}}
    {{end}}
  {{end}}
{{end}}
```
## Edit compendium message
Command : `change`
Usage : `$change <id> <edit>`

```go
{{$chan := 788017425215651850}}
{{if .CmdArgs}}
  {{$id := index .CmdArgs 0}}
  {{$msg := getMessage $chan $id}}
  {{if $msg.Embeds }}
    {{$embed := structToSdict (index $msg.Embeds 0) }}
    {{ range $k, $v := $embed }}
      {{- if eq (kindOf $v true) "struct" }}
        {{- $embed.Set $k (structToSdict $v) }}
      {{- end -}}
    {{ end }}
    {{if $embed.Footer.Text}}
      {{$user := reFind .User.Username $embed.Footer.Text}}
      {{if $user}}
        {{if ge (len .CmdArgs) 1}}
          {{$desc := joinStr "" (slice .CmdArgs 1)}}
          {{$embed.Set "description" $desc }}
          {{ if $embed.Footer }} 
            {{ $embed.Footer.Set "Icon_URL" $embed.Footer.IconURL }} 
          {{ end }}
          {{$embed = cembed $embed}}
          {{editMessage 788017425215651850 $id (complexMessageEdit "content" "" "embed" $embed)}}
        {{end}}
      {{end}}
    {{end}}
  {{end}}
{{end}}
```

