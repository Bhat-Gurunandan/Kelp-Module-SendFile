# NAME

Kelp::Module::SendFile - Dancer-style file sending with Kelp

# SYNOPSIS

    use Kelp::Module::SendFile;

# DESCRIPTION

Kelp::Module::SendFile enables returning a file from a route handler while 
controlling various aspects of the delivery

# METHODS

## send\_file

Lets the current route handler send a file to the client. 

    sub some_route_handler {
    	

       	# Do your processing
           ... 	
           return send_file(params->{file});
       }
    

Send file supports streaming possibility using PSGI streaming. The server should
support it but normal streaming is supported on most, if not all.
 

    sub your_route_handler{
    	

    	# Process...
    	

           return send_file( params->{file}, streaming => 1 );
       }
    

You can control the delivery using `callback`s.
 

First, `around_content` allows you to get the writer object and the chunk of
content read, and then decide what to do with each chunk:
 

    sub some_route_handler {

           return send_file(
               $file,
               streaming => 1,
               callbacks => {
                   around_content => sub {
                       my ( $writer, $chunk ) = @_;
                       $writer->write("* $chunk");
                   },
               },
           );
       }
    

You can use `around` to all get all the content (whether a filehandle if it's
a regular file or a full string if it's a scalar ref) and decide what to do with
it:
 

    sub some_route_handler {

        # Process ...
        

           return send_file(
               $file,
               streaming => 1,
               callbacks => {
                   around => sub {
                       my ( $writer, $content ) = @_;
    

                       # we know it's a text file, so we'll just stream
                       # line by line
                       while ( my $line = <$content> ) {
                           $writer->write($line);
                       }
                   },
               },
           );
       }
    

Or you could use `override` to control the entire streaming callback request:
 

    sub some_route_handler {
    	

    	# Process ...

           return send_file(
               $file,
               streaming => 1,
               callbacks => {
                   override => sub {
                       my ( $responder, $response ) = @_;
    

                       my $writer = $responder->( [ $newstatus, $newheaders ] );
                       $writer->write("some line");
                   },
               },
           );
       }
    

You can also set the number of bytes that will be read at a time (default being
42K bytes) using `bytes`:
 

    sub some_route_handler {

           return send_file(
               params->{file},
               streaming => 1,
               bytes     => 524288, # 512K
           );
       };
    

    

The content-type will be set depending on the current MIME types definition
 

If your filename does not have an extension, or you need to force a
specific mime type, you can pass it to `send_file` as follows:
 

       return send_file($file, content_type => 'image/png');
    

If you have your data in a scalar variable, `send_file` can be useful
as well. Pass a reference to that scalar, and `send_file` will behave
as if there was a file with that contents:
 

      return send_file( \$data, content_type => 'image/png' );
    

Note that this module is unable to guess the content type from the data
contents. Therefore you might need to set the `content_type`
properly. For this kind of usage an attribute named `filename` can be
useful.  It is used as the Content-Disposition header, to hint the
brower about the filename it should use.
 

      return send_file( \$data, content_type => 'image/png'
                                filename     => 'onion.png' );
    

# AUTHOR

Gurunandan R. Bhat <gbhat@pobox.com>

# COPYRIGHT

Copyright 2013- Gurunandan R. Bhat

# LICENSE

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.

# SEE ALSO
